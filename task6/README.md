# Task 6 — Docker
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Angular](https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)

---

## Репозитории


Backend: 
https://github.com/Aston-DevOps-Course/kanban-backend

Frontend: 
https://github.com/Aston-DevOps-Course/kanban-frontend

---

# Сборка backend

Перейти в репозиторий backend:

```bash
cd kanban-backend
````

## Сборка Docker image

```bash
docker build -t kanban-backend .
```

## Запуск отдельно (опционально)

```bash
docker run -p 8081:8081 kanban-backend
```

---

# Сборка frontend

Перейти в репозиторий frontend:

```bash
cd kanban-frontend
```

## Сборка Docker image

```bash
docker build -t kanban-frontend .
```

##  Запуск отдельно (опционально)

```bash
docker run -p 80:80 kanban-frontend
```

---

# Запуск через Docker Compose (основной способ)

Перейти в папку task6:

```bash
cd task6
```

## Запуск всей системы

```bash
docker compose up --build
```

---

# Доступ к приложению

Frontend (через балансировщик):

```
http://localhost:8080
```

Backend API:

```
http://localhost:8081
```

---

# Архитектура (Load Balancing)

Frontend работает в 2 экземплярах:

* frontend1
* frontend2

Перед ними стоит Nginx load balancer, который распределяет запросы между инстансами.

---

# Docker Compose структура

Сервисы:

* backend (Spring Boot)
* frontend1 (Angular + Nginx)
* frontend2 (Angular + Nginx)
* nginx (load balancer)

---

# Порядок запуска

1. backend поднимается после базы данных
2. frontend1 / frontend2 запускаются после backend
3. nginx стартует после frontend контейнеров

---

# Использованные best practices

## Backend:

* multi-stage build (build + runtime)
* lightweight JDK image
* non-root user
* clean separation build/runtime

## Frontend:

* Angular build stage (Node.js)
* production static serving через Nginx
* minimal nginx image
* no dev server in production

## Infrastructure:

* docker-compose orchestration
* load balancing через nginx
* горизонтальное масштабирование frontend (2 инстанса)

---
