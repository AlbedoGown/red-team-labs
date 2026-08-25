# Red Team Labs — Write-ups

Практические write-up'ы по лабораторным работам (HTB Academy) — разведка инфраструктуры, эксплуатация неправильных конфигураций сервисов, повышение привилегий через credential reuse и утечки секретов, а также эксплуатация уязвимостей веб-приложений через Metasploit Framework.

⚠️ Все флаги, пароли и хеши в этих write-up'ах — заменены на нереальные значения и не соответствуют реальным ответам лабораторных заданий.

## Содержание

| Лаба | Уровень | Основной вектор |
|---|---|---|
| [footprinting-easy.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-easy.md) | Easy | DNS zone transfer (AXFR) → subdomain enum → FTP → SSH key theft |
| [footprinting-medium.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-medium.md) | Medium | NFS anonymous mount → SMB → credential reuse → MSSQL |
| [footprinting-hard.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-hard.md) | Hard | SNMP community bruteforce → process argument leak → IMAP email → MySQL |
| [metasploit-fortilogger.md](https://github.com/AlbedoGown/red-team-labs/blob/main/metasploit-fortilogger.md) | Metasploit | Arbitrary file upload RCE (FortiLogger) → SYSTEM shell → NTLM hashdump |
| [starting-point-responder.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-responder.md) | Starting Point — Easy | LFI → UNC path (//attacker_ip) → NTLM capture (Responder) → hash crack → WinRM (Administrator) |
| [starting-point-three.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-three.md) | Starting Point — Easy | Vhost enum → S3/localstack anonymous write access → PHP web shell → reverse shell |
| [starting-point-cap.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-cap.md) | Starting Point — Easy | Dashboard IDOR (pcap leak) → FTP plaintext creds → Python cap_setuid privesc |

## Формат write-up'а

Каждый файл разбит на секции:

- **Recon** — что смотрели и почему.
- **Vulnerability / Misconfig** — в чём суть проблемы (root cause, не только "как эксплуатировали").
- **Exploitation** — шаги атаки.
- **Remediation** — как закрыть проблему в реальной инфраструктуре.
- **Lessons learned** — обобщение, применимое за пределами конкретной лабы.
