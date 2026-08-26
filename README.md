# Red Team Labs — Обзоры

Практические описания по лабораторным работам (Академия HTB) — разведка, эксплуатация неправильных конфигураций сервисов, повышение привилегий за счет повторного использования учетных данных и утечки секретов, а также эксплуатация уязвимостей веб-приложений через Metasploit Framework.

> ⚠️ Все флаги, пароли и хеши в описаниях — заменены на нереальные значения и не соответствуют реальным ответам лабораторных заданий.

## Содержание

| Файл | Уровень | Цепочка атаки |
|---|---|---|
| [footprinting-easy.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-easy.md) | Легкий | Передача зоны DNS (AXFR) → перечисление поддоменов → FTP → кража ключа SSH |
| [footprinting-medium.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-medium.md) | Середина | Анонимное монтирование NFS → SMB → повторное использование учетных данных → MSSQL |
| [footprinting-hard.md](https://github.com/AlbedoGown/red-team-labs/blob/main/footprinting-hard.md) | Жесткий | Перебор паролей в сообществе SNMP → утечка аргументов процесса → электронная почта IMAP → MySQL |
| [metasploit-fortilogger.md](https://github.com/AlbedoGown/red-team-labs/blob/main/metasploit-fortilogger.md) | Metasploit | Произвольная загрузка файла RCE (FortiLogger) → Системная оболочка → Хэшдамп NTLM |
| [starting-point-responder.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-responder.md) | Начальная точка — Легко | LFI → UNC-путь (//attacker_ip) → захват NTLM (Responder) → взлом хеша → WinRM (Administrator) |
| [starting-point-three.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-three.md) | Начальная точка — Легко | Перечисление Vhost → Анонимный доступ на запись в S3/localstack → Веб-оболочка PHP → Обратная оболочка |
| [starting-point-cap.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-cap.md) | Начальная точка — Легко | Панель управления IDOR (утечка pcap) → Учетные данные FTP в открытом тексте → Привилегии Python cap_setuid |
| [starting-point-funnel.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-funnel.md) | Отправная точка — Уровень 1 | Анонимная утечка FTP → распыление паролей (Hydra) → переадресация локальных портов SSH (PostgreSQL) |
| [starting-point-bike.md](https://github.com/AlbedoGown/red-team-labs/blob/main/starting-point-bike.md) | Начальная точка — Уровень 2 | Handlebars SSTI → небезопасный контекст выполнения Node.js → влияние на корневом уровне |
| starting-point-tactics.md | Starting Point — Tier 1 | Blank Administrator Password → SMB Administrative Shares (C$) Access |

## Формат write-up'а

Каждый файл разбит по разделам:

- Recon — что смотрели и почему.
- Уязвимость/неверная конфигурация — в чём суть проблемы (основная причина, не только «какатировали»).
- Эксплуатация — шаги нападения.
- Исправление — как закрыть проблему в существующей инфраструктуре.
- Извлеченные уроки — обобщение, применение в отношении конкретных лабораторий.