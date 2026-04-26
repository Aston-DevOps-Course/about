# Модуль 3

## 1. Создаем системного пользователя для сервиса

Запускаем:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appuser
```

---

## 2. Создаем скрипт

Размещаем скрипт, например:

```bash
sudo mkdir -p /usr/local/bin
sudo nano /usr/local/bin/app_logger.sh
```

Содержимое:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

APP_DIR="/opt/app"
LOG_FILE="${APP_DIR}/log.txt"

mkdir -p "$APP_DIR"
touch "$LOG_FILE"

chown appuser:appuser "$APP_DIR" "$LOG_FILE"
chmod 750 "$APP_DIR"
chmod 640 "$LOG_FILE"

generate_random_string() {
    tr -dc 'A-Za-z0-9' </dev/urandom | head -c $(( RANDOM % 20 + 1 ))
}

while true
do
    generate_random_string >> "$LOG_FILE"
    printf '\n' >> "$LOG_FILE"
    sleep 17
done
```

Сделать исполняемым:

```bash
sudo chmod +x /usr/local/bin/app_logger.sh
```

---

## 3. systemd unit для автозагрузки

Создаем сервис:

```bash
sudo nano /etc/systemd/system/app-logger.service
```

Содержимое:

```ini
[Unit]
Description=Random String Logger
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
ExecStart=/usr/local/bin/app_logger.sh
Restart=always
RestartSec=5

# Hardening (best practices)
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

---

## 4. Включаем автозапуск

Применяем:

```bash
sudo systemctl daemon-reload
sudo systemctl enable app-logger.service
sudo systemctl start app-logger.service
```

Проверяем:

```bash
sudo systemctl status app-logger.service
```

Проверяем лог:

```bash
tail -f /opt/app/log.txt
```

После перезагрузки сервис стартует автоматически.

Проверка:

```bash
sudo reboot
```

После перезагрузки:

```bash
systemctl status app-logger
```

---

## 5. Настраиваем logrotate

Создаем конфиг:

```bash
sudo nano /etc/logrotate.d/app_logger
```

Содержимое:

```conf
/opt/app/log.txt {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
    create 0640 appuser appuser
}
```

Описание:
- `daily` — ротация ежедневно
- `rotate 14` — хранить 14 архивов
- `compress` — сжатие
- `delaycompress` — сжимать со следующей ротации
- `copytruncate` — безопасно для процесса, который держит открытый лог
- `missingok` — не ошибка, если лог отсутствует
- `notifempty` — не ротировать пустой файл

---

## 6. Проверка logrotate

Проверка конфигурации:

```bash
sudo logrotate -d /etc/logrotate.d/app_logger
```

Принудительный тест:

```bash
sudo logrotate -f /etc/logrotate.d/app_logger
```

Проверка:

```bash
ls -lah /opt/app
```

Должны появиться:
- `log.txt`
- архивы вида:
- `log.txt.1`
- `log.txt.2.gz`

---

## Итоговая структура

```bash
/usr/local/bin/app_logger.sh
/etc/systemd/system/app-logger.service
/etc/logrotate.d/app_logger
/opt/app/log.txt
```

---

Использованы:
- `set -Eeuo pipefail`
- отдельный сервисный пользователь
- `systemd` вместо rc.local/cron @reboot
- systemd hardening
- ротация через logrotate
- принцип least privilege
- автоматический restart при падении процесса