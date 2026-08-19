# Footprinting — Medium: WINMEDIUM (NFS/SMB/MSSQL/RDP chain)

## Задача
Найти пароль пользователя `HTB` в таблице БД `devsacc`.

## 1. Recon (Nmap)

```
sudo nmap -A 10.129.202.41
```

Ключевые находки:

| Порт | Сервис |
|------|--------|
| 111/2049 | NFS (rpcbind, mountd, nlockmgr) |
| 135/139/445 | Windows SMB/RPC |
| 3389 | RDP (Windows Server, hostname WINMEDIUM) |

**Что здесь интересно:** NFS на Windows-хосте — нетипичная комбинация (обычно NFS = *nix). Значит, вероятно, установлен Services for NFS на Windows Server, что часто настраивают неаккуратно, без ограничений экспорта.

## 2. NFS enumeration

```
showmount -e 10.129.202.41
```

Результат: `/TechSupport (everyone)`.

**Root cause:** экспорт NFS с правом доступа `everyone` — то есть без ограничения по IP/сети. Правильная практика — явно указывать разрешённые хосты/подсети в конфигурации экспорта:

```
/TechSupport 10.129.0.0/16(ro,root_squash)
```

вместо открытого доступа всем. Это классическая ошибка **excessive permissions / no access control list**.

## 3. Монтирование и поиск данных

```
sudo mkdir NFS && sudo mount -t nfs 10.129.202.41:/TechSupport ./NFS
sudo ls -lA NFS/
```

Почти все тикет-файлы пустые (0 байт) — кроме одного с реальным содержимым (1305 байт). Обычный recon-приём: **сортировка файлов по размеру**, а не только по имени, чтобы быстро отсеять "пустышки".

```
sudo cat NFS/ticket4238791283782.txt
```

Внутри — переписка тех-поддержки, где пользователь `alex` вставил конфиг SMTP с паролем в открытом виде — `alex:<PASSWORD_PLACEHOLDER>`.

**Ключевой урок:** утечка credentials через service desk / support-переписку — очень частый вектор в реальных инцидентах. Пользователи вставляют конфиги "для примера", не замечая, что там пароль.

## 4. SMB enumeration с найденными credentials

```
smbclient -L //10.129.202.41 -U alex
```

Список ресурсов включает нестандартный `devshare` (помимо системных ADMIN$, C$, IPC$, Users).

```
smbclient //10.129.202.41/devshare -U alex
get important.txt
awk 1 important.txt
```

Внутри — `sa:<PASSWORD_PLACEHOLDER>` — похоже на creds учётки MSSQL `sa` (system administrator в SQL Server).

## 5. Password reuse → RDP как Administrator

Гипотеза: пароль от `sa` мог быть переиспользован для локального `Administrator` в Windows.

```
xfreerdp /v:10.129.202.41 /u:Administrator /p:'<PASSWORD_PLACEHOLDER>' /dynamic-resolution
```

Успешный логин подтвердил гипотезу.

**Термин:** это классический **credential reuse / password reuse** — одна из самых частых причин horizontal & vertical movement в корпоративных сетях. Компрометация одного сервиса (SQL Server) даёт доступ к ОС целиком, потому что администратор использовал один и тот же пароль для разных учётных записей и систем.

## 6. Извлечение финального пароля из MSSQL

После RDP — открыли SQL Server Management Studio, подключились к локальному инстансу и выполнили:

```sql
SELECT * FROM devsacc WHERE name='HTB';
```

Пароль: `FAKE_MEDIUM_PASSWORD_qwerty000` *(заменено на нереальное значение)*

## Remediation

- Ограничивать NFS-экспорты конкретными IP/подсетями, использовать `root_squash`.
- Обучать сотрудников не вставлять реальные пароли в тикеты поддержки; внедрить DLP-сканирование вложений service desk.
- Запретить повторное использование паролей между сервисными аккаунтами (SQL `sa`) и доменными/локальными административными учётками.
- Отключать или сильно ограничивать `sa` в MSSQL, использовать Windows Authentication вместо Mixed Mode там, где возможно.
- Мониторить RDP-логины административных аккаунтов (алерты на аномальные подключения).

## Lessons learned

- Сортировка файлов по размеру в общих ресурсах — быстрый способ найти "иголку в стоге сена".
- Всегда проверять credential reuse между разными типами сервисов (DB ↔ OS ↔ AD) — это один из самых недооценённых, но самых результативных векторов privilege escalation.
- NFS `everyone`-экспорты — почти гарантированная находка при пентесте инфраструктуры смешанного типа (Windows + *nix сервисы).
