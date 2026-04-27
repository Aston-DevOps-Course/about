# Модуль 4

## 1. Установка Nginx

Обновляем пакеты и устанавливаем nginx:

```bash
sudo apt update
sudo apt install -y nginx
```

Проверяем запуск:

```bash
systemctl status nginx
```

Включаем автозапуск:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 2. Настройка локального домена

### 2.1 Добавляем запись в hosts

Редактируем файл:

```bash
sudo nano /etc/hosts
```

Добавляем строку:

```
127.0.0.1 app.local
```

---

## 3. Создание конфигурации Nginx для app.local с self-signed сертификатом

### 3.1 Создаем сертификат

```bash
sudo mkdir -p /etc/nginx/ssl
```

```bash
sudo openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/nginx/ssl/app.local.key \
-out /etc/nginx/ssl/app.local.crt \
-subj "/C=DE/L=Frankfurt/O=LocalDev/CN=app.local"
```


### 3.2 Создаем конфиг:

```bash
sudo nano /etc/nginx/sites-available/app.local
```

Содержимое:

```nginx
server {
    listen 80;
    server_name app.local;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name app.local;

    ssl_certificate /etc/nginx/ssl/app.local.crt;
    ssl_certificate_key /etc/nginx/ssl/app.local.key;

    root /var/www/app.local;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### 3.1 Создаем директорию сайта

```bash
sudo mkdir -p /var/www/app.local
```

Создаем тестовую страницу:

```bash
echo "app.local is working" | sudo tee /var/www/app.local/index.html
```

---

### 3.2 Активируем сайт

```bash
sudo ln -s /etc/nginx/sites-available/app.local /etc/nginx/sites-enabled/
```

Проверяем конфигурацию:

```bash
sudo nginx -t
```

Перезапускаем nginx:

```bash
sudo systemctl reload nginx
```

---

## 4. Скрипт проверки доступности ресурса

Создаем скрипт:

```bash
sudo nano /usr/local/bin/check_resource.sh
```

### Bash версия

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

URL="${1:-}"

if [[ -z "$URL" ]]; then
    echo "Usage: $0 <url>"
    exit 2
fi

HTTP_CODE=$(curl -o /dev/null -s -w "%{http_code}" --max-time 5 "$URL" || true)

if [[ "$HTTP_CODE" -ge 200 && "$HTTP_CODE" -lt 400 ]]; then
    echo "OK: $URL is reachable (HTTP $HTTP_CODE)"
    exit 0
else
    echo "ERROR: $URL is NOT reachable (HTTP $HTTP_CODE)"
    exit 1
fi
```

Сделать исполняемым:

```bash
sudo chmod +x /usr/local/bin/check_resource.sh
```

---

## 5. Примеры запуска

### Успешный кейс:

```bash
check_resource.sh https://app.local
```

---

### Ошибка (например, несуществующий домен):

```bash
check_resource.sh http://nonexistent.local
```

---

## Итоговая структура

```
/etc/nginx/sites-available/app.local
/etc/nginx/sites-enabled/app.local
/etc/hosts
/etc/nginx/ssl/app.local.crt
/etc/nginx/ssl/app.local.key
/var/www/app.local/index.html
/usr/local/bin/check_resource.sh
```