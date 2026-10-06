# Telegram Subscription Platform

![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![Vue](https://img.shields.io/badge/Vue-3-4FC08D?logo=vuedotjs&logoColor=white)
![Tests](https://img.shields.io/badge/tests-965%20passed-brightgreen)
![Coverage](https://img.shields.io/badge/coverage-93%25-brightgreen)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/license-Proprietary%20Commercial-orange)

> Production-oriented Telegram subscription platform with Telegram Mini App,
> Telegram Stars payments, access management, admin panel and background workers.

Telegram-платформа для продажи подписок на закрытые Telegram-ресурсы.

Проект объединяет Telegram Mini App, Telegram Bot, оплату через Telegram Stars, управление подписками, контроль доступа к Telegram-ресурсам, административную панель и фоновые workers.

Основная задача системы — автоматически провести пользователя по цепочке:

```text
Telegram
   ↓
WebApp
   ↓
Авторизация пользователя
   ↓
Выбор плана
   ↓
Telegram Stars
   ↓
Подтверждение платежа
   ↓
Подписка
   ↓
User Access
   ↓
Telegram Resource
   ↓
Синхронизация доступа
```

Проект построен как переиспользуемая backend/frontend-основа для платных Telegram-сообществ, каналов, групп, курсов и других сервисов с подписочной моделью.

---

# Содержание

* [Возможности](#возможности)
* [Демонстрация](#демонстрация)
* [Скриншоты](#скриншоты)
* [Архитектура](#архитектура)
* [Dependency Injection](#dependency-injection)
* [Технологический стек](#технологический-стек)
* [Структура проекта](#структура-проекта)
* [Backend](#backend)
* [Frontend](#frontend)
* [Требования](#требования)
* [Конфигурация](#конфигурация)
* [Переменные окружения](#переменные-окружения)
* [Важное замечание о секретах](#важное-замечание-о-секретах)
* [Быстрый запуск](#быстрый-запуск)
* [Проверка состояния](#проверка-состояния)
* [Миграции базы данных](#миграции-базы-данных)
* [Demo-данные](#demo-данные)
* [Очистка базы](#очистка-базы)
* [Настройка Telegram Bot](#настройка-telegram-bot)
* [Telegram Mini App](#telegram-mini-app)
* [URL Telegram WebApp](#url-telegram-webapp)
* [Платежный поток](#платежный-поток)
* [Идемпотентность платежей](#идемпотентность-платежей)
* [Подписки](#подписки)
* [План и ресурсы](#план-и-ресурсы)
* [Активация и деактивация ресурсов](#активация-и-деактивация-ресурсов)
* [Effective Access](#effective-access)
* [Telegram Access Synchronization](#telegram-access-synchronization)
* [Join Requests](#join-requests)
* [Transactional Outbox](#transactional-outbox)
* [Workers](#workers)
* [Worker Review](#worker-review)
* [Административная панель](#административная-панель)
* [API](#api)
* [Frontend API](#frontend-api)
* [Тестирование](#тестирование)
* [Структура тестов](#структура-тестов)
* [Качество кода](#качество-кода)
* [Полная проверка перед commit](#полная-проверка-перед-commit)
* [Локальная разработка Backend](#локальная-разработка-backend)
* [Frontend Development](#frontend-development)
* [Database Development](#database-development)
* [Production](#production)
* [Docker Services](#docker-services)
* [Security](#security)
* [Известные ограничения](#известные-ограничения)
* [Roadmap](#roadmap)
* [Статистика проекта](#статистика-проекта)
* [Основные бизнес-сценарии](#основные-бизнес-сценарии)
* [Почему проект разделён на слои](#почему-проект-разделён-на-слои)
* [Основные директории](#основные-директории)
* [Полезные команды](#полезные-команды)
* [Лицензия](#лицензия)
* [Контакты](#контакты)
* [Project Status](#project-status)

---

# Возможности

## Telegram

* Telegram Bot на `aiogram`
* Telegram Mini App
* авторизация через Telegram WebApp `initData`
* проверка подлинности Telegram WebApp данных на backend
* обработка join requests
* автоматическое одобрение или отклонение заявок на вступление
* управление членством пользователей
* ban/unban пользователей
* создание и отзыв Telegram invite links
* синхронизация фактического доступа пользователя с состоянием подписки
* защита Telegram administrators и resource owners от случайного удаления при синхронизации

---

## Подписки

Система поддерживает:

* тарифные планы;
* стоимость и длительность плана;
* привязку Telegram-ресурсов к плану;
* создание подписки;
* продление подписки;
* автоматическое истечение подписки;
* получение списка подписок пользователя;
* просмотр конкретной подписки;
* административный просмотр подписок.

Текущая модель статусов подписки:

```text
ACTIVE
EXPIRED
```

Отдельного статуса `CANCELLED` в текущей версии нет.

---

## Telegram Stars

Платежная система построена вокруг Telegram Stars.

Поддерживается:

* создание платежа;
* создание Telegram invoice;
* получение invoice link;
* подтверждение Telegram payment;
* проверка checkout;
* проверка суммы;
* проверка валюты;
* проверка charge ID;
* идемпотентная обработка платежей;
* создание подписки после успешной оплаты;
* создание access;
* генерация outbox events;
* уведомление пользователя.

Используемая валюта:

```text
XTR
```

---

## Управление доступом

Доступ пользователя определяется не только наличием подписки.

Система учитывает:

```text
User
 +
Subscription
 +
UserAccess
 +
Plan
 +
Resource
```

Поддерживаются:

* выдача доступа;
* отзыв доступа;
* проверка effective access;
* список доступов пользователя;
* административный просмотр доступов;
* access invites;
* синхронизация доступа с Telegram.

---

## Access Invites

Для Telegram-ресурсов поддерживаются access invites.

Система позволяет:

* создавать invite;
* синхронизировать invite;
* отзывать invite;
* связывать invite с пользователем и ресурсом;
* автоматически управлять жизненным циклом invite.

---

## Административная панель

Администратор может управлять:

* пользователями;
* планами;
* ресурсами;
* ресурсами внутри планов;
* подписками;
* платежами;
* доступами;
* dashboard.

Поддерживается отдельная административная авторизация.

---

## Background Workers

Для фоновых операций используется:

* Redis;
* ARQ;
* transactional outbox;
* отдельный worker runtime.

В фоновые задачи вынесены, в частности:

* синхронизация Telegram access;
* retry Telegram synchronization;
* access invite synchronization;
* expiration subscriptions;
* уведомления об активации подписки;
* уведомления об истечении подписки;
* уведомления об успешном платеже.

---

# Демонстрация

## Видео

### User Interface Overview

[![User Interface Overview — Plans, Subscriptions, Access & Profile](https://img.youtube.com/vi/6D6HJmTkYY4/maxresdefault.jpg)](https://youtu.be/6D6HJmTkYY4)

**User Interface Overview — Plans, Subscriptions, Access & Profile**

Обзор пользовательского интерфейса Telegram Mini App:

* главная страница;
* тарифные планы;
* детали тарифа;
* подписки;
* доступы;
* профиль пользователя.

[Смотреть видео на YouTube](https://youtu.be/6D6HJmTkYY4)

---

### Admin Panel Overview

[![Admin Panel Overview — Plans, Resources & Management](https://img.youtube.com/vi/m5v2PQB6iU4/maxresdefault.jpg)](https://youtu.be/m5v2PQB6iU4)

**Admin Panel Overview — Plans, Resources & Management**

Обзор административной панели:

* Dashboard;
* пользователи;
* планы;
* ресурсы;
* подписки;
* платежи;
* доступы;
* создание ресурсов;
* создание тарифных планов;
* управление ресурсами и планами.

[Смотреть видео на YouTube](https://youtu.be/m5v2PQB6iU4)

---

### User Subscription Flow

[![User Subscription Flow — From Plan Selection to Access](https://img.youtube.com/vi/iglSHNKuXAw/maxresdefault.jpg)](https://youtu.be/iglSHNKuXAw)

**User Subscription Flow — From Plan Selection to Access**

Демонстрация основного пользовательского сценария:

```text
Открытие Telegram Mini App
        ↓
Авторизация
        ↓
Выбор плана
        ↓
Просмотр деталей плана
        ↓
Оплата через Telegram Stars
        ↓
Подтверждение платежа
        ↓
Создание подписки
        ↓
Выдача доступа
        ↓
Проверка доступа
```

[Смотреть видео на YouTube](https://youtu.be/iglSHNKuXAw)


---

# Скриншоты

## Пользовательский интерфейс

### Авторизация

![Авторизация](assets/auth.png)

### Главная страница

![Главная страница](assets/user-home.png)

### Список планов

![Список планов](assets/user-plans.png)

### Детали плана

![Детали плана](assets/user-plan-details.png)

### Подписки

![Подписки](assets/user-subscriptions.png)

### Доступы

![Доступы](assets/user-access.png)

---

## Административная панель

### Dashboard

![Dashboard](assets/admin-dashboard.png)

### Пользователи

![Пользователи](assets/admin-users.png)

### Планы

![Планы](assets/admin-plans.png)

### Ресурсы

![Ресурсы](assets/admin-resources.png)

### Подписки

![Подписки](assets/admin-subscriptions.png)

### Платежи

![Платежи](assets/admin-payments.png)

### Доступы

![Доступы](assets/admin-access.png)

---

## Telegram Bot

К примеру:

### Уведомление об успешной оплате

![Уведомление об успешной оплате](assets/tg_bot_notif_succ_payment.png)


---

# Архитектура

Проект построен на layered architecture с разделением `domain`, `application`, `infrastructure` и `interfaces`.

```text
┌─────────────────────────────────────────────┐
│                 Interfaces                  │
│                                             │
│       FastAPI        Telegram Bot           │
│          │                 │                │
└──────────┼─────────────────┼────────────────┘
           │                 │
           ▼                 ▼
┌─────────────────────────────────────────────┐
│                Application                  │
│                                             │
│  Use Cases / Services / Ports / ReadModels  │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│                  Domain                     │
│                                             │
│ User / Plan / Resource / Subscription       │
│ Payment / Access / AccessInvite             │
└──────────────────────┬──────────────────────┘
                       │
                       │ implemented by
                       ▼
┌─────────────────────────────────────────────┐
│              Infrastructure                 │
│                                             │
│ PostgreSQL / Redis / Telegram / ARQ         │
└─────────────────────────────────────────────┘
```

Основное правило зависимости:

```text
Interfaces
    ↓
Application
    ↓
Domain

Infrastructure
    ↑
implements application ports
```

Domain и application layers не должны зависеть от FastAPI, SQLAlchemy, Redis или aiogram.

---

# Dependency Injection

Сборка приложения выполняется через composition root.

Основная точка композиции:

```text
backend/app/container.py
```

`Container` отвечает за связывание:

* repositories;
* application services;
* Telegram adapters;
* Redis;
* transaction manager;
* workers;
* notification services;
* application dependencies.

Это позволяет использовать реальные инфраструктурные реализации в production и заменять их тестовыми реализациями в unit tests.

---

# Технологический стек

| Компонент         | Технология                       |
| ----------------- | -------------------------------- |
| Backend           | Python 3.13                      |
| API               | FastAPI                          |
| Validation        | Pydantic 2                       |
| ORM               | SQLAlchemy 2                     |
| Database          | PostgreSQL 18                    |
| Database driver   | asyncpg                          |
| Migrations        | Alembic                          |
| Telegram          | aiogram 3                        |
| Payments          | Telegram Stars                   |
| Redis             | Redis 8                          |
| Background jobs   | ARQ                              |
| Authentication    | Telegram WebApp + Redis sessions |
| Password hashing  | Argon2                           |
| Frontend          | Vue 3                            |
| Frontend language | TypeScript                       |
| Frontend build    | Vite                             |
| Testing           | pytest                           |
| Coverage          | pytest-cov                       |
| Linting           | Ruff                             |
| Type checking     | mypy                             |
| Containers        | Docker Compose                   |

Основные backend-зависимости зафиксированы в `pyproject.toml`.

---

# Структура проекта

```text
telegram_subscription_platform/

│
├── backend/
│   ├── app/
│   ├── application/
│   ├── domain/
│   ├── infrastructure/
│   ├── interfaces/
│   └── workers/
│
├── frontend/
│   └── src/
│
├── migrations/
│   └── versions/
│
├── scripts/
│   ├── clear_db.py
│   ├── seed_demo.py
│   └── worker_review.py
│
├── tests/
│   ├── app/
│   ├── integration/
│   ├── unit/
│   └── workers/
│
├── docker/
│   ├── backend/
│   ├── frontend/
│   └── worker/
│
├── docker-compose.yml
├── pyproject.toml
├── alembic.ini
├── outbox.md
├── LICENSE
├── .env.example
└── README.md
```

---

# Backend

Backend разделён на несколько слоёв.

## `backend/domain`

Содержит бизнес-модель:

```text
domain/

├── access/
├── access_invite/
├── payment/
├── plan/
├── resource/
├── subscription/
├── subscription_resource/
└── user/
```

В domain находятся:

* entities;
* domain rules;
* domain errors;
* repository interfaces.

---

## `backend/application`

Application layer содержит use cases.

Основные области:

```text
application/

├── access/
├── admin/
├── auth/
├── payments/
├── plans/
├── resources/
├── subscriptions/
├── subscription_resources/
├── telegram/
└── users/
```

Примеры use cases:

```text
CreatePlan
UpdatePlan
DeactivatePlan
ActivateResource
CreatePayment
ProcessPayment
CreateTelegramInvoice
CreateSubscription
ExpireSubscription
CreateAccessInvite
RevokeAccess
SynchronizeResourceAccess
LoginViaTelegramWebApp
```

---

## `backend/infrastructure`

Содержит реализации портов:

```text
infrastructure/

├── auth/
├── database/
├── jobs/
├── notifications/
└── telegram/
```

В частности:

* SQLAlchemy repositories;
* PostgreSQL models;
* Redis sessions;
* Telegram adapters;
* ARQ queue;
* outbox dispatcher;
* notification adapters.

---

## `backend/interfaces`

Содержит внешние интерфейсы приложения:

```text
interfaces/

├── api/
└── telegram/
```

API реализован через FastAPI.

Telegram interface реализован через aiogram.

---

# Frontend

Frontend построен на Vue 3 + TypeScript.

Основная структура:

```text
frontend/src/

├── app/
├── features/
├── layouts/
├── pages/
├── router/
├── services/
├── stores/
├── styles/
└── types/
```

Feature-oriented структура позволяет изолировать бизнес-функциональность.

Основные frontend features:

```text
access
admin
auth
payments
plans
resources
subscriptions
users
```

Есть отдельные layouts:

```text
UserLayout
AdminLayout
```

И отдельные маршруты для:

```text
User
Admin
```

---

# Требования

Для запуска через Docker:

* Docker;
* Docker Compose;
* Git.

Для локальной разработки backend:

* Python 3.13;
* virtual environment;
* PostgreSQL/Redis либо Docker services.

Для frontend-разработки:

* Node.js;
* npm.

---

# Конфигурация

В репозитории находится:

```text
.env.example
```

Для Docker Compose используется отдельный файл:

```text
.env.docker
```

`docker-compose.yml` передаёт контейнерам значения из `.env.docker`:

```yaml
env_file:
  - .env.docker
```

При этом Python `Settings` по умолчанию загружает:

```text
.env
```

Таким образом:

```text
.env
    ↓
локальный запуск Python

.env.docker
    ↓
Docker Compose containers
```

Для Docker Compose рекомендуется создать `.env.docker` на основе `.env.example`:

```powershell
Copy-Item .env.example .env.docker
```

---

# Переменные окружения

Актуальный шаблон переменных окружения:

```env
APP_NAME=Telegram Subscription Platform
APP_VERSION=0.1.0
DEBUG=false

POSTGRES_DB=telegram_subscription_platform
POSTGRES_USER=telegram_subscription_platform
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_PORT=5432

DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform
TEST_DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform_test

REDIS_PORT=6379
REDIS_URL=redis://redis:6379/0

BACKEND_PORT=8000
FRONTEND_PORT=5173

ADMIN_LOGIN=admin
ADMIN_PASSWORD_HASH=your_argon2id_password_hash
ADMIN_SESSION_TTL=86400

USER_SESSION_TTL=86400

TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_WEBAPP_URL=https://your-domain.com
TELEGRAM_WEBAPP_MAX_AGE_SECONDS=900
TELEGRAM_WEBAPP_MAX_INIT_DATA_BYTES=8192
```

Параметры соответствуют `backend/app/config.py`.

## Application

```env
APP_NAME=Telegram Subscription Platform
APP_VERSION=0.1.0
DEBUG=false
```

* `APP_NAME` — название приложения.
* `APP_VERSION` — версия приложения.
* `DEBUG` — режим отладки.

## PostgreSQL

```env
POSTGRES_DB=telegram_subscription_platform
POSTGRES_USER=telegram_subscription_platform
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_PORT=5432

DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform
TEST_DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform_test
```

* `POSTGRES_DB` — основная база данных.
* `POSTGRES_USER` — пользователь PostgreSQL.
* `POSTGRES_PASSWORD` — пароль PostgreSQL.
* `POSTGRES_PORT` — порт PostgreSQL.
* `DATABASE_URL` — connection string основной базы.
* `TEST_DATABASE_URL` — connection string тестовой базы.

## Redis

```env
REDIS_PORT=6379
REDIS_URL=redis://redis:6379/0
```

Redis используется для:

* пользовательских sessions;
* admin sessions;
* ARQ;
* background jobs;
* outbox dispatch.

## Backend / Frontend

```env
BACKEND_PORT=8000
FRONTEND_PORT=5173
```

Эти параметры используются для соответствующих сервисов Docker Compose.

## Admin

```env
ADMIN_LOGIN=admin
ADMIN_PASSWORD_HASH=your_argon2id_password_hash
ADMIN_SESSION_TTL=86400
```

`ADMIN_PASSWORD_HASH` должен содержать Argon2id hash, а не обычный пароль.

## User authentication

```env
USER_SESSION_TTL=86400
```

Определяет срок жизни пользовательской application session.

## Telegram

```env
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_WEBAPP_URL=https://your-domain.com
TELEGRAM_WEBAPP_MAX_AGE_SECONDS=900
TELEGRAM_WEBAPP_MAX_INIT_DATA_BYTES=8192
```

* `TELEGRAM_BOT_TOKEN` — token Telegram Bot.
* `TELEGRAM_WEBAPP_URL` — публичный HTTPS URL Telegram Mini App.
* `TELEGRAM_WEBAPP_MAX_AGE_SECONDS` — максимальный допустимый возраст Telegram WebApp `initData`.
* `TELEGRAM_WEBAPP_MAX_INIT_DATA_BYTES` — максимальный размер входных WebApp данных.

---

# Важное замечание о секретах

Никогда не добавляйте в Git:

* `TELEGRAM_BOT_TOKEN`;
* production PostgreSQL password;
* production Redis credentials;
* admin password;
* production authentication secrets;
* другие production secrets.

В `.env.example` используются только примерные значения.

Реальные значения должны находиться только в локальных или production environment files:

```text
.env
.env.docker
```

Эти файлы не должны публиковаться в репозитории.

---

# Быстрый запуск

## 1. Клонирование

```bash
git clone https://github.com/idtmt/telegram_subscription_platform.git
cd telegram_subscription_platform
```

---

## 2. Создание Docker environment

Создайте:

```text
.env.docker
```

на основе:

```text
.env.example
```

Например:

```powershell
Copy-Item .env.example .env.docker
```

После этого замените placeholders на реальные значения.

Минимально необходимо настроить:

```env
POSTGRES_PASSWORD=your_postgres_password
DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform
TEST_DATABASE_URL=postgresql+asyncpg://telegram_subscription_platform:your_postgres_password@postgres:5432/telegram_subscription_platform_test

ADMIN_PASSWORD_HASH=your_argon2id_password_hash

TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_WEBAPP_URL=https://your-domain.com
```

Для Telegram Mini App `TELEGRAM_WEBAPP_URL` должен указывать на публичный HTTPS URL.

---

## 3. Запуск

```powershell
docker compose up --build
```

Docker Compose поднимет:

```text
postgres
redis
backend
worker
frontend
```

Архитектура сети:

```text
                ┌──────────────┐
                │   Frontend   │
                │    :5173     │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │   Backend    │
                │    :8000     │
                └───┬──────┬───┘
                    │      │
         ┌──────────┘      └──────────┐
         ▼                            ▼
  ┌─────────────┐              ┌─────────────┐
  │ PostgreSQL  │              │    Redis    │
  │    :5432    │              │    :6379    │
  └─────────────┘              └──────┬──────┘
                                      │
                                      ▼
                               ┌─────────────┐
                               │   Worker    │
                               └─────────────┘
```

---

# Проверка состояния

После запуска:

```powershell
docker compose ps
```

Backend health endpoint:

```text
http://localhost:8000/health
```

Ожидаемый ответ:

```json
{
  "status": "ok"
}
```

Frontend:

```text
http://localhost:5173
```

FastAPI Swagger:

```text
http://localhost:8000/docs
```

OpenAPI schema:

```text
http://localhost:8000/openapi.json
```

---

# Миграции базы данных

После запуска контейнеров примените миграции:

```powershell
docker compose exec backend alembic upgrade head
```

Текущая цепочка миграций:

```text
6ad0e012458b
create initial database schema
        ↓
c0ba61f17250
add outbox table
        ↓
90fa81f53e5c
add access invites table
        ↓
6fd949c0ab35
remove cancelled subscription status
```

Проверить текущую ревизию:

```powershell
docker compose exec backend alembic current
```

Применить все миграции:

```powershell
docker compose exec backend alembic upgrade head
```

Откатить последнюю миграцию:

```powershell
docker compose exec backend alembic downgrade -1
```

---

# Demo-данные

Для демонстрации предусмотрен deterministic seed.

Запуск:

```powershell
docker compose exec backend python scripts/seed_demo.py
```

Seed создаёт:

```text
Users:          5
Plans:          3
Resources:      3
Subscriptions: 6
Payments:       7
User accesses:  9
Invites:        3
Outbox events:  0
```

Demo-сценарий содержит несколько различных состояний пользователей.

Пример:

```text
Alex

└── Premium ACTIVE
    ├── Basic access
    └── Premium access


Maria

└── Basic ACTIVE
    └── Basic access


Ivan

├── Premium EXPIRED
└── Basic EXPIRED


Denis

├── Basic EXPIRED
└── Premium ACTIVE
    ├── Basic access
    └── Premium access


Sofia

└── No subscription
```

Это позволяет сразу проверить различные состояния интерфейса.

---

# Очистка базы

Для удаления application data используется:

```powershell
docker compose exec backend python scripts/clear_db.py
```

Скрипт очищает application tables, но не удаляет:

* Alembic migrations;
* database schema;
* Docker volumes.

После очистки demo-данные можно создать заново:

```powershell
docker compose exec backend python scripts/seed_demo.py
```

---

# Настройка Telegram Bot

Создайте Telegram Bot через `@BotFather`.

Полученный token укажите:

```env
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
```

Бот должен иметь необходимые административные права в Telegram-ресурсах, которыми управляет платформа.

В зависимости от типа ресурса боту могут потребоваться права на:

* обработку join requests;
* управление участниками;
* ban/unban;
* создание invite links;
* отзыв invite links.

---

# Telegram Mini App

Frontend может запускаться как Telegram Mini App.

Telegram передаёт приложению:

```text
Telegram WebApp initData
```

Backend самостоятельно валидирует эти данные.

Общий flow:

```text
Telegram
    ↓
Mini App
    ↓
initData
    ↓
Backend validation
    ↓
TelegramWebAppUser
    ↓
User lookup / creation
    ↓
Application session
```

Нельзя считать данные пользователя из frontend доверенными до проверки на backend.

---

# URL Telegram WebApp

URL Mini App хранится в конфигурации окружения.

Основная переменная:

```env
TELEGRAM_WEBAPP_URL=https://your-domain.com
```

Backend загружает значение через `Settings` из:

```text
backend/app/config.py
```

Telegram keyboard использует настроенный URL:

```python
WebAppInfo(
    url=settings.telegram_webapp_url,
)
```

URL не должен быть захардкожен в исходном коде.

Для development-сценария публичный HTTPS URL может быть предоставлен через tunnel, например Cloudflare Tunnel.

В production необходимо использовать собственный HTTPS domain.

Не следует коммитить временный development tunnel URL как production configuration.

---

# Платежный поток

Основной payment flow:

```text
User
 │
 │ select plan
 ▼
Create payment
 │
 ▼
Create Telegram invoice
 │
 ▼
Telegram Stars
 │
 ▼
Telegram checkout
 │
 ▼
Validate checkout
 │
 ▼
Confirm payment
 │
 ▼
ProcessPayment
 │
 ├── Payment
 │
 ├── Subscription
 │
 ├── UserAccess
 │
 └── Outbox events
 │
 ▼
Worker
 │
 ├── Telegram synchronization
 ├── expiration scheduling
 └── notification
```

---

# Идемпотентность платежей

Payment processing рассчитан на повторную доставку одного и того же события.

Обрабатываются ограничения:

* уникальность payment payload;
* уникальность Telegram charge ID;
* проверка валюты;
* проверка суммы;
* проверка длительности;
* повторная обработка успешного платежа.

Повторная обработка уже успешно завершённого платежа не должна создавать вторую подписку или повторно выдавать доступ.

---

# Подписки

Текущая модель:

```text
ACTIVE
EXPIRED
```

Жизненный цикл:

```text
ACTIVE
   │
   │ expiration
   ▼
EXPIRED
```

Пользовательские endpoints:

```http
GET /user/subscriptions
GET /user/subscriptions/{subscription_id}
POST /user/subscriptions/{subscription_id}/renew
```

В текущей версии отдельного endpoint для отмены подписки нет.

Автоматического recurring renewal в текущей модели также нет.

---

# План и ресурсы

План может включать несколько Telegram-ресурсов.

Например:

```text
Basic
└── Basic Resource

Premium
├── Basic Resource
└── Premium Resource

VIP
├── Basic Resource
├── Premium Resource
└── VIP Resource
```

При активации подписки пользователь получает доступ к активным ресурсам соответствующего плана.

---

# Активация и деактивация ресурсов

Для ресурса предусмотрены:

```http
POST /admin/resources/{resource_id}/activate
DELETE /admin/resources/{resource_id}
```

При деактивации ресурс удаляется из планов.

После повторной активации ресурс не добавляется автоматически обратно в старые планы — управление связями выполняется отдельно.

---

# Effective Access

Фактический доступ пользователя определяется application/domain rules.

Упрощённо:

```text
UserAccess exists
       +
UserAccess is ACTIVE
       +
Subscription is effective
       +
Resource is ACTIVE
       ↓
Effective Access
```

При отсутствии effective access пользователь не должен получать доступ к соответствующему Telegram resource.

---

# Telegram Access Synchronization

При изменении состояния доступа система может создать outbox event.

Worker выполняет:

```text
Database state
      ↓
User effective access
      ↓
Telegram resource
      ↓
Member status
      ↓
ban / unban / synchronization
```

Особое правило:

Telegram administrators и resource owners не должны быть случайно удалены механизмом пользовательской синхронизации доступа.

---

# Join Requests

При поступлении Telegram join request:

```text
Telegram
   ↓
Join Request
   ↓
HandleJoinRequest
   ↓
User exists?
   ↓
Resource exists?
   ↓
Effective access?
   ├── YES → approve
   └── NO  → decline
```

---

# Transactional Outbox

Внешние операции не должны выполняться внутри критической database transaction, если их результат нельзя атомарно сохранить вместе с данными PostgreSQL.

Вместо этого используется transactional outbox.

```text
┌───────────────────────────────┐
│        DB Transaction         │
│                               │
│  Payment                      │
│  Subscription                 │
│  UserAccess                   │
│  OutboxEvent                  │
│                               │
└───────────────┬───────────────┘
                │ commit
                ▼
        Outbox Dispatcher
                │
                ▼
             Redis
                │
                ▼
             ARQ Job
                │
                ▼
       External side effect
```

Таким образом database state и информация о необходимой фоновой операции сохраняются в одной транзакции.

Подробное описание outbox находится в:

```text
outbox.md
```

---

# Workers

Worker запускается отдельным контейнером:

```text
worker
```

Основные задачи:

```text
telegram-sync
retry-telegram-sync
access-invite-sync
expire-subscription
subscription-activated-notification
subscription-expired-notification
payment-succeeded-notification
```

Worker использует:

```text
PostgreSQL
Redis
ARQ
Telegram Bot API
```

---

# Worker Review

Для development предусмотрен:

```text
scripts/worker_review.py
```

Получить список команд:

```powershell
docker compose exec worker python scripts/worker_review.py --help
```

Команды:

```text
telegram-sync
retry-telegram-sync
access-invite-sync
expire-subscription
subscription-activated-notification
subscription-expired-notification
payment-succeeded-notification
```

Эти команды предназначены для проверки и ручного запуска отдельных worker jobs при разработке и диагностике.

---

# Административная панель

Admin interface доступен через frontend.

Основные разделы:

```text
Dashboard
Users
Plans
Resources
Subscriptions
Payments
Access
```

Администратор может:

* создавать планы;
* изменять планы;
* деактивировать планы;
* создавать ресурсы;
* изменять ресурсы;
* активировать ресурсы;
* деактивировать ресурсы;
* добавлять ресурсы в планы;
* удалять ресурсы из планов;
* просматривать пользователей;
* просматривать подписки;
* просматривать платежи;
* просматривать access records.

---

# API

API реализован через FastAPI.

После запуска:

```text
http://localhost:8000/docs
```

OpenAPI:

```text
http://localhost:8000/openapi.json
```

Основные группы endpoints:

```text
/auth
/user
/admin
```

Актуальный API contract следует проверять через OpenAPI после запуска приложения.

---

## User API

Включает операции для:

* авторизации;
* профиля;
* планов;
* платежей;
* подписок;
* access.

Примеры:

```http
GET /user/plans

GET /user/subscriptions

GET /user/subscriptions/{subscription_id}

POST /user/subscriptions/{subscription_id}/renew

GET /user/access

GET /user/payments
```

---

## Admin API

Включает операции для:

* dashboard;
* users;
* plans;
* resources;
* plan resources;
* subscriptions;
* payments;
* access.

Примеры:

```http
GET /admin/dashboard

GET /admin/users

GET /admin/users/{user_id}

GET /admin/plans

POST /admin/plans

GET /admin/plans/{plan_id}

PATCH /admin/plans/{plan_id}

GET /admin/resources

POST /admin/resources

POST /admin/resources/{resource_id}/activate

DELETE /admin/resources/{resource_id}
```

Полный актуальный API contract следует смотреть через OpenAPI:

```text
/docs
```

---

# Frontend API

Frontend взаимодействует с backend API через отдельные services и stores.

Основные пользовательские области:

```text
auth
plans
subscriptions
payments
access
users
```

Основные административные области:

```text
admin
plans
resources
subscriptions
payments
access
users
```

API contract не дублируется полностью в README, поскольку источником истины является OpenAPI schema backend.

---

# Frontend

Frontend:

```text
Vue 3
TypeScript
Vite
Vue Router
```

Основные feature modules:

```text
features/

├── access/
├── admin/
├── auth/
├── payments/
├── plans/
├── resources/
├── subscriptions/
└── users/
```

Основные layouts:

```text
AdminLayout.vue
UserLayout.vue
```

Основные страницы пользователя:

```text
Home
Plans
Plan Details
Subscriptions
Subscription Details
Payments
Access
Profile
```

Основные административные страницы:

```text
Login
Dashboard
Users
User Details
Plans
Plan Create
Plan Details
Plan Edit
Resources
Resource Details
Subscriptions
Payments
Access
```

---

# Тестирование

Проект содержит unit и integration tests.

Запустить весь набор:

```powershell
pytest
```

С coverage:

```powershell
pytest --cov=backend --cov-report=term-missing
```

Текущий baseline проекта:

```text
965 passed
0 failed
93% coverage
```

Последняя полная проверка:

```text
ruff format --check .
373 files already formatted

ruff check .
All checks passed!

mypy .
Success: no issues found in 371 source files

pytest --cov=backend --cov-report=term-missing

965 passed

TOTAL 4905 statements
333 missed
93% coverage
```

---

# Структура тестов

```text
tests/

├── app/
│
├── integration/
│   ├── database/
│   └── repositories/
│
├── unit/
│   ├── application/
│   ├── domain/
│   ├── infrastructure/
│   └── interfaces/
│
└── workers/
```

Покрываются:

* domain rules;
* application use cases;
* repositories;
* database constraints;
* transactions;
* Telegram adapters;
* authentication;
* API dependencies;
* Telegram middleware;
* workers;
* outbox;
* payment processing;
* subscription lifecycle;
* access management.

---

# Качество кода

Проект использует Ruff и mypy.

## Ruff

Проверка:

```powershell
ruff check .
```

Форматирование:

```powershell
ruff format .
```

Проверка форматирования без изменения файлов:

```powershell
ruff format --check .
```

---

## mypy

Проект использует strict type checking:

```powershell
mypy .
```

Конфигурация находится в:

```text
pyproject.toml
```

---

# Полная проверка перед commit

Рекомендуемый набор:

```powershell
ruff format --check .
ruff check .
mypy .
pytest --cov=backend --cov-report=term-missing
```

Если все команды завершились успешно, backend проходит текущий quality baseline проекта.

---

# Локальная разработка Backend

Создать virtual environment:

```powershell
py -3.13 -m venv .venv
```

Активировать:

```powershell
.\.venv\Scripts\Activate.ps1
```

Установить проект с development dependencies:

```powershell
pip install -e ".[dev]"
```

После этого можно запускать:

```powershell
pytest

ruff check .

ruff format --check .

mypy .
```

Для локального запуска Python `Settings` использует `.env`.

---

# Frontend Development

Перейти в frontend:

```powershell
cd frontend
```

Установить зависимости:

```powershell
npm install
```

Запустить development server:

```powershell
npm run dev
```

Production build:

```powershell
npm run build
```

Перед production deployment необходимо проверить frontend environment variables.

---

# Database Development

Для создания новой migration:

```powershell
alembic revision --autogenerate -m "describe change"
```

После генерации migration необходимо обязательно проверить файл вручную.

Применить:

```powershell
alembic upgrade head
```

Проверить состояние:

```powershell
alembic current
```

---

# Production

Docker Compose в репозитории предназначен прежде всего для development/demo окружения.

Перед production deployment необходимо отдельно настроить:

* HTTPS;
* reverse proxy;
* production domain;
* Telegram Mini App URL;
* secrets management;
* PostgreSQL backups;
* Redis persistence;
* monitoring;
* logging;
* alerting;
* restart policies;
* resource limits;
* firewall;
* database access restrictions;
* Redis access restrictions;
* production CORS;
* production frontend environment;
* Telegram bot production runtime.

Не следует считать текущий `docker-compose.yml` полностью hardened production deployment configuration.

---

# Docker Services

Текущий Compose содержит:

```text
postgres
redis
backend
worker
frontend
```

## PostgreSQL

```text
postgres:18-alpine
```

Используется для хранения application data.

---

## Redis

```text
redis:8-alpine
```

Используется для:

* sessions;
* ARQ;
* background jobs;
* outbox dispatch.

Включён AOF:

```text
--appendonly yes
```

---

## Backend

FastAPI application.

Health check:

```text
GET /health
```

Container port:

```text
8000
```

---

## Worker

Отдельный ARQ worker runtime.

Worker использует тот же application container/configuration foundation, что и backend, но запускает background processing вместо HTTP server.

---

## Frontend

Vue application, собранная через Vite и отдаваемая через nginx.

Container port:

```text
80
```

На host по умолчанию:

```text
5173
```

---

# Security

Перед использованием проекта в production необходимо:

1. заменить все development secrets;
2. установить новый Telegram Bot token;
3. установить уникальный PostgreSQL password;
4. установить production admin credentials;
5. использовать HTTPS;
6. не публиковать PostgreSQL наружу без необходимости;
7. не публиковать Redis наружу без необходимости;
8. настроить firewall;
9. настроить backups;
10. настроить monitoring;
11. не хранить `.env` в Git.

Особенно важно не использовать development credentials из документации или примеров.

---

# Известные ограничения

Текущая версия является production-oriented foundation, но не является автоматически полностью готовой production deployment системой.

В частности, production deployment требует отдельной настройки:

* CI/CD;
* monitoring;
* centralized logging;
* backups;
* secrets management;
* reverse proxy;
* HTTPS;
* production Telegram configuration;
* infrastructure security;
* operational alerting.

Кроме того, часть Telegram и внешних API сценариев зависит от реального окружения Telegram и прав Bot.

---

# Roadmap

Возможные следующие направления развития:

* CI/CD;
* production deployment templates;
* webhook-based Telegram runtime;
* monitoring;
* metrics;
* structured logging;
* payment reconciliation;
* automated reconciliation worker;
* расширенная аналитика;
* subscription statistics;
* дополнительные E2E tests;
* frontend E2E tests;
* расширенные admin reports.

---

# Статистика проекта

Текущий baseline версии:

```text
Git tracked files: 488

Python files:       333

Vue components:      55

TypeScript files:    34

Total code:       ~55,900 LOC

Backend:          ~11,300 LOC

Frontend:         ~22,500 LOC

Tests:            ~20,200 LOC

Tests:                965

Coverage:              93%
```

По состоянию на текущий baseline:

```text
ruff format --check .  → passed
ruff check .           → passed
mypy .                 → passed
pytest                 → 965 passed
```

Статистика является снимком конкретной версии проекта и может изменяться по мере развития репозитория.

---

# Основные бизнес-сценарии

## Покупка подписки

```text
User
  ↓
Open Mini App
  ↓
Login via Telegram
  ↓
Open Plans
  ↓
Select Plan
  ↓
Create Payment
  ↓
Create Telegram Invoice
  ↓
Pay with Telegram Stars
  ↓
Confirm Payment
  ↓
Create/activate Subscription
  ↓
Create/activate UserAccess
  ↓
Outbox Event
  ↓
Worker
  ↓
Telegram Access Synchronization
```

---

## Истечение подписки

```text
ACTIVE Subscription
        ↓
expiration time
        ↓
ExpireSubscription
        ↓
EXPIRED
        ↓
Outbox events
        ↓
Worker
        ↓
Access synchronization
        ↓
Telegram access revoked
```

---

## Join Request

```text
Telegram Join Request
        ↓
HandleJoinRequest
        ↓
Check User
        ↓
Check Resource
        ↓
Check Effective Access
        │
        ├── true  → approve
        │
        └── false → decline
```

---

# Почему проект разделён на слои

Такое разделение позволяет изменять инфраструктуру без переписывания бизнес-логики.

Например:

```text
FastAPI
    ↓
Application
    ↓
Domain
```

не зависит от конкретного:

```text
PostgreSQL
Redis
aiogram
ARQ
```

Инфраструктурные компоненты подключаются через ports/adapters.

Это также позволяет тестировать бизнес-логику без реального Telegram API и без необходимости поднимать всю инфраструктуру для каждого unit test.

---

# Основные директории

| Директория                    | Назначение                                   |
| ----------------------------- | -------------------------------------------- |
| `backend/app`                 | application configuration и composition root |
| `backend/domain`              | domain model и business rules                |
| `backend/application`         | use cases и application services             |
| `backend/infrastructure`      | PostgreSQL, Redis, Telegram, jobs            |
| `backend/interfaces/api`      | FastAPI API                                  |
| `backend/interfaces/telegram` | Telegram Bot interface                       |
| `backend/workers`             | worker runtime                               |
| `frontend`                    | Vue Mini App и Admin UI                      |
| `migrations`                  | Alembic migrations                           |
| `scripts`                     | demo и development utilities                 |
| `tests`                       | unit/integration/worker tests                |

---

# Полезные команды

## Запуск

```powershell
docker compose up --build
```

## Остановка

```powershell
docker compose down
```

## Логи backend

```powershell
docker compose logs -f backend
```

## Логи worker

```powershell
docker compose logs -f worker
```

## Логи frontend

```powershell
docker compose logs -f frontend
```

## Логи PostgreSQL

```powershell
docker compose logs -f postgres
```

## Логи Redis

```powershell
docker compose logs -f redis
```

## Проверка контейнеров

```powershell
docker compose ps
```

## Миграции

```powershell
docker compose exec backend alembic upgrade head
```

## Demo

```powershell
docker compose exec backend python scripts/seed_demo.py
```

## Очистка application data

```powershell
docker compose exec backend python scripts/clear_db.py
```

## Worker review

```powershell
docker compose exec worker python scripts/worker_review.py --help
```

## Backend tests

```powershell
pytest
```

## Coverage

```powershell
pytest --cov=backend --cov-report=term-missing
```

## Lint

```powershell
ruff check .
```

## Format check

```powershell
ruff format --check .
```

## Type check

```powershell
mypy .
```

---

# Лицензия

Проект распространяется по **Proprietary Commercial License**.

Полный текст лицензии находится в:

```text
LICENSE
```

Copyright:

```text
Copyright (c) 2026 idtmt

All rights reserved.
```

Стандартная лицензия предоставляет покупателю право:

* использовать исходный код;
* запускать проект на собственной инфраструктуре;
* использовать проект в коммерческих целях;
* изменять и расширять исходный код;
* создавать на его основе коммерческое приложение или сервис;
* использовать проект в рамках одного разрешённого коммерческого проекта;
* использовать скомпилированную, собранную или развёрнутую версию проекта в рамках разрешённого проекта.

Без отдельного разрешения запрещается:

* перепродавать исходный код;
* распространять исходный код как шаблон;
* публиковать полный проект или существенные части исходного кода;
* загружать проект или существенные части исходного кода в публичные репозитории;
* передавать лицензию третьим лицам;
* сублицензировать проект;
* продавать доступ к исходному коду;
* создавать конкурирующий коммерческий template, boilerplate или starter kit;
* использовать проект для создания другого продукта, предназначенного для независимой повторной продажи исходного кода.

Стандартная лицензия действует для одного независимого коммерческого проекта.

Для дополнительных проектов, агентских сценариев, multi-project использования или передачи исходного кода клиенту требуется соответствующая расширенная лицензия.

Использование проекта для разработки продукта или сервиса для клиента разрешено в рамках приобретённой лицензии, однако исходный код самого шаблона не может быть передан клиенту без соответствующего разрешения.

Third-party libraries, frameworks и другие компоненты сохраняют свои собственные лицензии.

Техническая поддержка, сопровождение, hosting, deployment, обновления и будущие версии не входят в стандартную лицензию, если иное прямо не согласовано отдельно.

---

# Контакты

**Telegram:** [@idtmt](https://t.me/idtmt)

**Email:** [idtmt.dev@gmail.com](mailto:idtmt.dev@gmail.com)

По вопросам приобретения, лицензирования и проекта используйте Telegram или email.

---

# Project Status

Проект представляет собой готовую переиспользуемую основу для разработки Telegram subscription platform.

Текущая версия включает:

```text
✓ FastAPI backend

✓ Layered architecture

✓ Domain/Application separation

✓ Ports & adapters

✓ PostgreSQL

✓ SQLAlchemy

✓ Alembic

✓ Redis

✓ ARQ workers

✓ Transactional Outbox

✓ Telegram Bot

✓ Telegram WebApp authentication

✓ Telegram Stars payments

✓ Subscription management

✓ Resource management

✓ Resource activation/deactivation

✓ Access management

✓ Access invites

✓ Telegram membership synchronization

✓ Admin authentication

✓ Admin panel

✓ User Mini App

✓ Environment-based Telegram WebApp URL

✓ Demo seed

✓ Database cleanup utility

✓ Worker review utilities

✓ Unit tests

✓ Integration tests

✓ Static type checking

✓ Ruff linting

✓ 93% backend test coverage
```

Основной запуск:

```powershell
docker compose up --build
```

После запуска:

```text
Frontend:

http://localhost:5173


Backend:

http://localhost:8000


Swagger:

http://localhost:8000/docs


Health:

http://localhost:8000/health
```

Для production необходимо отдельно настроить инфраструктуру, безопасность, HTTPS, secrets management, monitoring, backups и Telegram production configuration.

