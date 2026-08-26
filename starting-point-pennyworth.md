# HTB Starting Point — Pennyworth (Tier 2 / Easy)

> ⚠️ Все флаги, пароли, хеши, IP-адреса и иные чувствительные значения в этом write-up заменены на нереальные перед публикацией. Репозиторий публичный.

## 1. Recon

Первичное полное TCP-сканирование показало один доступный сервис:

| Port | Service | Identified technology |
|---|---|---|
| 8080/tcp | HTTP | Jetty 9.4.39.v20210325 |

Дополнительные HTTP-результаты:

- HTTP-заголовок указывал на `Jetty(9.4.39.v20210325)`.
- `robots.txt` содержал запрещённый путь `/`.
- Веб-интерфейс на `http://10.129.X.X:8080` оказался Jenkins.
- Технологическое fingerprinting-расширение определило Jenkins 2.289.1, Java и Jetty 9.4.39.

Логика разведки: единственный открытый веб-порт стал приоритетной поверхностью атаки. Версия Jenkins полезна для анализа потенциальных CVE, но сама по себе не доказывает эксплуатационный путь. Следующим шагом стала проверка доступных функций Jenkins и механизма аутентификации.

## 2. Vulnerability / Misconfig Analysis

Атака стала возможной из-за цепочки небезопасных конфигураций.

| Chain link | Root Cause | Security impact |
|---|---|---|
| Jenkins exposed on TCP/8080 | Административный CI/CD-интерфейс доступен по сети без дополнительного сетевого периметра | Увеличение внешней поверхности атаки |
| Weak administrative credentials | Для входа была принята предсказуемая учётная запись `root:<REDACTED_PASSWORD>` | Несанкционированный доступ к Jenkins |
| Access to Script Console | Скомпрометированная учётная запись имела административные функции Jenkins | Возможность выполнить произвольный Groovy-код |
| Over-privileged Jenkins process | Процесс Jenkins имел доступ к файлам, доступным root | RCE через Jenkins привело к root-level access |

После входа стала доступна функция **Manage Jenkins → Script Console**. Script Console предназначена для администрирования Jenkins и выполняет Groovy-код в контексте процесса Jenkins.

Важно: Groovy сам по себе не выдаёт root-привилегии. Код наследует права ОС-процесса Jenkins. В этой лаборатории возможность прочитать `/root/flag.txt` подтверждает, что Jenkins выполнялся с чрезмерными привилегиями.

## 3. Exploitation

1. Провести полное TCP-сканирование и обнаружить HTTP-сервис на 8080/tcp.
2. Выполнить HTTP-enumeration: проверить заголовки, `robots.txt` и страницу приложения.
3. Идентифицировать Jenkins и проверить механизм входа на наличие слабых/дефолтных учётных данных.
4. После аутентификации открыть **Manage Jenkins → Script Console**.
5. Использовать Groovy-код, который запускает дочерний shell-процесс и устанавливает исходящее TCP-соединение к контролируемому listener.

Минималистичный шаблон Java/Groovy-логики:

```groovy
String host = "<ATTACKER_IP>"
int port = <ATTACKER_PORT>
String cmd = "/bin/bash"

Process process = new ProcessBuilder(cmd)
    .redirectErrorStream(true)
    .start()

// Connect process input/output to a controlled TCP listener.
```

6. Получить интерактивную shell-сессию в контексте Jenkins.
7. Проверить уровень доступа и получить root-флаг:

```text
/root/flag.txt
<REDACTED_FLAG>
```


### Reference

- [Reverse Shell Cheat Sheet — Internal All The Things](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#ognl)

## 4. Remediation & SOC Detection

### Remediation

- Не публиковать Jenkins controller напрямую в интернет; ограничить доступ через VPN, bastion host, reverse proxy и IP allowlist.
- Заменить слабые и дефолтные учётные данные на уникальные длинные пароли; включить SSO/MFA.
- Ограничить административные права Jenkins по принципу least privilege и регулярно проверять матрицу ролей.
- Ограничить доступ к Script Console только строго необходимым доверенным администраторам; аудитировать каждое её использование.
- Запускать Jenkins от выделенного непривилегированного сервисного аккаунта, никогда не от `root`.
- Использовать изолированные build agents/containers и минимальные файловые, сетевые и credential-права.
- Ограничить исходящий трафик Jenkins только необходимыми репозиториями, registries и внутренними сервисами.
- Поддерживать Jenkins, плагины, Java runtime и базовую ОС в актуальном состоянии.

### Detection opportunities

| Indicator | Data source | Detection idea | Severity |
|---|---|---|---|
| Серия неуспешных попыток входа в Jenkins | Jenkins access/audit logs, reverse proxy, WAF | Multiple failed POST requests to login endpoint from one source or distributed sources | Medium/High |
| Успешный admin login с нового IP | Jenkins audit logs, IdP, VPN logs | New geo/IP/device for privileged Jenkins account | High |
| Открытие или запрос к Script Console | Jenkins audit/access logs | Alert on access to administrative script-execution functionality | High |
| Java/Jenkins запускает shell | EDR, auditd, Sysmon for Linux | Parent-child relationship: `java` or Jenkins process → `sh`, `bash`, `python`, `nc`, `curl`, `wget` | Critical |
| Необычный исходящий TCP-трафик Jenkins | Firewall, NetFlow, EDR | Long-lived outbound session from Jenkins to an unapproved IP/port | High |
| CI-сервис читает root-only files | auditd, EDR, eBPF telemetry | Jenkins/Java accesses `/root/` or other privileged file paths | Critical |

Пример поведенческой логики EDR:

```text
IF parent_process IN ("java", "jenkins")
AND child_process IN ("sh", "bash", "nc", "ncat", "curl", "wget", "python")
THEN raise high-priority alert
```

## 5. Lessons Learned

- Сканирование определяет поверхность атаки, но не заменяет enumeration и анализ конфигурации приложения.
- `robots.txt` не защищает ресурсы: он лишь просит поисковые системы не индексировать указанные пути и может раскрывать интересные endpoints.
- Jenkins Script Console — высокорисковая административная функция, поскольку способна выполнять произвольный код на хосте.
- Компрометация Jenkins admin не должна автоматически означать компрометацию ОС: разделение прав Jenkins и ОС является критичным контролем.
- CI/CD-инфраструктура особенно ценна для атакующих: она часто имеет доступ к исходному коду, секретам, артефактам, cloud credentials и внутренним сетевым ресурсам.
- Для SOC важен не только вход в Jenkins, но и последующая поведенческая цепочка: admin login → Script Console → Java spawns shell → anomalous outbound connection.

---

> **Authorized-lab notice:** This write-up documents activity in the Hack The Box Starting Point laboratory environment only.
