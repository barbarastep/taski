# Taski

Приложение для управления задачами: создание, редактирование, удаление
и отметка о выполнении.

Развёрнутый проект: https://taski25698.duckdns.org/

Исходный проект предоставлен Яндекс Практикумом.
Контейнеризация и развёртывание — [barbarastep](https://github.com/barbarastep).

## Технологии

Python 3.12, Django, Django REST Framework, Gunicorn,
React, PostgreSQL 13, Nginx, Docker и Docker Compose.

## Структура проекта

- `backend/` — Django API и Dockerfile бэкенда.
- `frontend/` — React-приложение и Dockerfile фронтенда.
- `nginx/` — Dockerfile и конфигурация внутреннего Nginx.
- `docker-compose.yml` — конфигурация четырёх сервисов.
- `.env.example` — пример переменных окружения.

Сервисы Docker Compose:

- `db` — база данных PostgreSQL.
- `backend` — API под управлением Gunicorn.
- `frontend` — копирование собранного фронтенда в общий том.
- `gateway` — раздача фронтенда и проксирование запросов к API.

Данные PostgreSQL сохраняются в томе `pg_data`,
фронтенд и статика Django — в томе `static`.
Сервис `frontend` завершает работу с кодом `0` после копирования файлов.

## Локальный запуск на Mac

Необходимы Git и запущенный Docker Desktop с Docker Compose.
Все команды выполняются в терминале Mac.

### 1. Подготовка проекта

```bash
git clone https://github.com/barbarastep/taski.git
cd taski
cp .env.example .env
```

Если репозиторий уже клонирован, перейдите в его директорию.

В `.env` замените:

- `POSTGRES_PASSWORD` — на собственный пароль базы данных;
- `SECRET_KEY` — на случайный секретный ключ Django.

Для генерации ключа можно выполнить:

```bash
openssl rand -hex 32
```

Для локального запуска оставьте:

```dotenv
DB_HOST=db
DB_PORT=5432
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1
```

Файл `.env` содержит секреты и не должен попадать в Git.

### 2. Сборка образов и запуск базы

```bash
docker compose build
docker compose up -d --wait db
```

### 3. Миграции и статика Django

```bash
docker compose run --rm backend python manage.py migrate --noinput
```

```bash
docker compose run --rm backend sh -c 'python manage.py collectstatic --noinput && mkdir -p /static/backend_static && cp -r /app/collected_static/. /static/backend_static/'
```

### 4. Запуск приложения

```bash
docker compose up -d
```

Приложение доступно по адресу http://localhost:9001/.

### 5. Проверка

```bash
docker compose ps -a
curl -i http://localhost:9001/api/tasks/
```

API должен вернуть `200 OK` и JSON со списком задач.

## Остановка

Из директории проекта:

```bash
docker compose down
```

Именованные тома с данными сохраняются.

## Развёртывание на сервере

На сервере используются те же сервисы Docker Compose.
Внутренний gateway доступен на `127.0.0.1:9001`.

Внешний Nginx принимает запросы к `taski25698.duckdns.org`
и передаёт их на этот порт. HTTPS настроен через Certbot
и сертификат Let's Encrypt.

Домен добавлен в `ALLOWED_HOSTS` серверного `.env`.
Серверный `.env` создаётся отдельно и не хранится в репозитории.
