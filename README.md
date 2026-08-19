# Red Team Labs — Write-ups

Практические write-up'ы по лабораторным работам (HTB Academy) — разведка инфраструктуры, эксплуатация неправильных конфигураций сервисов, повышение привилегий через credential reuse и утечки секретов, а также эксплуатация уязвимостей веб-приложений через Metasploit Framework.

⚠️ Все флаги, пароли и хеши в этих write-up'ах — **заменены на нереальные значения** и не соответствуют реальным ответам лабораторных заданий.

## Содержание

| Лаба | Уровень | Основной вектор |
|------|---------|------------------|
| [footprinting-easy.md](footprinting-easy.md) | Easy | DNS zone transfer (AXFR) → subdomain enum → FTP → SSH key theft |
| [footprinting-medium.md](footprinting-medium.md) | Medium | NFS anonymous mount → SMB → credential reuse → MSSQL |
| [footprinting-hard.md](footprinting-hard.md) | Hard | SNMP community bruteforce → process argument leak → IMAP email → MySQL |
| [metasploit-fortilogger.md](metasploit-fortilogger.md) | Metasploit | Arbitrary file upload RCE (FortiLogger) → SYSTEM shell → NTLM hashdump |

## Формат write-up'а

Каждый файл разбит на секции:

1. **Recon** — что смотрели и почему.
2. **Vulnerability / Misconfig** — в чём суть проблемы (root cause, не только "как эксплуатировали").
3. **Exploitation** — шаги атаки.
4. **Remediation** — как закрыть проблему в реальной инфраструктуре.
5. **Lessons learned** — обобщение, применимое за пределами конкретной лабы.
