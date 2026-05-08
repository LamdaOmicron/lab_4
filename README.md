# Лабораторные работы №2, №3, №4: RESTful API с аутентификацией и Swagger документацией

## Описание проекта

RESTful API для управления игровыми персонажами (на примере Pathfinder 2e) с полной JWT аутентификацией, OAuth поддержкой (Yandex/VK), и автоматической Swagger документацией. Реализован полный CRUD с мягким удалением (soft delete) и пагинацией. Проект использует Django + Django REST Framework, PostgreSQL в Docker.

## Технологии

- Python 3.12
- Django 4.2
- Django REST Framework 3.14
- drf-spectacular (OpenAPI/Swagger)
- PostgreSQL 16
- Docker & Docker Compose
- JWT аутентификация (Access + Refresh токены)
- OAuth 2.0 (Yandex, VK)
- bcrypt для хеширования паролей

## Запуск проекта

### Предварительные требования

- Установленный Docker и Docker Compose
- (Опционально) Git для клонирования репозитория

### Шаги для запуска

1. **Клонировать репозиторий**:
   ```bash
   git clone <url-репозитория>
   cd lab_5
   ```

2. **Создать файл `.env`** на основе `.env.example` (см. ниже).

3. **Запустить контейнеры**:
   ```bash
   docker-compose up --build
   ```
   - При первом запуске автоматически выполнятся миграции.
   - API будет доступно по адресу: `http://localhost:4200/api/characters/`
   - **Swagger документация** доступна по адресу: `http://localhost:4200/api/docs/` (только в режиме разработки)

4. **Остановить контейнеры**:
   ```bash
   docker-compose down
   ```

## Переменные окружения

Создайте файл `.env` в корне проекта со следующим содержимым:

```env
# PostgreSQL
DB_USER=student
DB_PASSWORD=student_secure_password
DB_NAME=wp_labs
DB_HOST=postgres
DB_PORT=5432

# Приложение
APP_PORT=4200

# Окружение (development или production)
NODE_ENV=development

# JWT Configuration
JWT_ACCESS_SECRET=super_secret_access_key_change_in_prod
JWT_REFRESH_SECRET=super_secret_refresh_key_change_in_prod
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=7d

# OAuth Provider Configuration
CLIENT_ID=your_client_id
CLIENT_SECRET=your_client_secret
CALLBACK_URL=http://localhost:4200/auth/yandex/callback
```

**Примечание:** файл `.env` не должен попадать в систему контроля версий (добавлен в `.gitignore`).

## API Endpoints

### Базовый URL: `http://localhost:4200`

#### Auth (Аутентификация)

| Метод | URI | Описание | Статус успеха | Требует авторизации |
|-------|-----|----------|---------------|---------------------|
| POST | `/auth/register/` | Регистрация нового пользователя | 201 Created | Нет |
| POST | `/auth/login/` | Вход пользователя (установка cookies) | 200 OK | Нет |
| POST | `/auth/refresh/` | Обновление JWT токенов | 200 OK | Нет (использует cookie) |
| GET | `/auth/whoami/` | Проверка авторизации | 200 OK | Да |
| POST | `/auth/logout/` | Выход из текущей сессии | 200 OK | Да |
| POST | `/auth/logout-all/` | Выход из всех сессий | 200 OK | Да |
| GET | `/auth/oauth/<provider>/` | Инициация OAuth входа | 200 OK | Нет |
| GET | `/auth/oauth/<provider>/callback/` | OAuth callback | 302 Redirect | Нет |
| POST | `/auth/forgot-password/` | Запрос сброса пароля | 200 OK | Нет |
| POST | `/auth/reset-password/` | Сброс пароля | 200 OK | Нет |

#### Characters (Персонажи)

Базовый URL: `http://localhost:4200/api/characters/`

| Метод | URI | Описание | Статус успеха | Требует авторизации |
|-------|-----|----------|---------------|---------------------|
| GET | `/` | Список персонажей (с пагинацией) | 200 OK | Нет |
| POST | `/` | Создание нового персонажа | 201 Created | Нет |
| GET | `/<uuid:id>/` | Получение персонажа по ID | 200 OK | Нет |
| PUT | `/<uuid:id>/` | Полное обновление персонажа | 200 OK | Нет |
| PATCH | `/<uuid:id>/` | Частичное обновление персонажа | 200 OK | Нет |
| DELETE | `/<uuid:id>/` | Мягкое удаление персонажа | 204 No Content | Нет |

### Пагинация

Параметры передаются в query string:
- `page` – номер страницы (по умолчанию 1)
- `limit` – количество записей на странице (по умолчанию 10, максимум 100)

## Swagger документация

Документация генерируется автоматически на основе кода приложения с использованием библиотеки `drf-spectacular`.

### Доступ к документации

- **Development режим** (`NODE_ENV=development`): Документация доступна по адресу `http://localhost:4200/api/docs/`
- **Production режим** (`NODE_ENV=production`): Документация **недоступна** (возвращает 404)

### Безопасность в документации

- Чувствительные данные (пароли, соли, refresh токены) скрыты из схем ответов
- Защищенные эндпоинты помечены значком замка
- Схема безопасности настроена для работы с cookie-based аутентификацией

## Структура проекта

```
lab_5/
├── characters/               # приложение персонажей
├── auth_app/                 # приложение аутентификации
├── users/                    # приложение пользователей
├── myproject/                # настройки Django
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── README.md
```

## Контрольные вопросы

### Лабораторная работа №3 (Аутентификация)

1. **В чем разница между аутентификацией и авторизацией?**
   - Аутентификация — проверка личности (кто вы?)
   - Авторизация — проверка прав доступа (что разрешено?)

2. **Что такое соль (salt)?**
   - Случайная строка, добавляемая к паролю перед хешированием
   - Гарантирует уникальность хешей даже для одинаковых паролей

3. **Из каких частей состоит JWT?**
   - Header, Payload, Signature

4. **Зачем хранить Refresh Token в БД?**
   - Для возможности отзыва токена (logout)
   - Для контроля активных сессий

5. **Преимущества HttpOnly cookies перед LocalStorage?**
   - Недоступны для JavaScript (защита от XSS)
   - Автоматическая отправка браузером

6. **Зачем нужен параметр state в OAuth 2.0?**
   - Защита от CSRF атак

7. **Шаги Authorization Code Grant:**
   1. Редирект на провайдера с client_id и state
   2. Авторизация у провайдера
   3. Возврат с authorization code
   4. Обмен code на access token
   5. Получение данных пользователя

8. **Разница между /logout и /logout-all?**
   - /logout отзывает текущий токен
   - /logout-all отзывает все токены пользователя

### Лабораторная работа №4 (Swagger)

1. **OpenAPI vs Swagger UI?**
   - OpenAPI — спецификация
   - Swagger UI — инструмент визуализации

2. **Code-First vs Design-First?**
   - Code-First: документация из кода (используется здесь)
   - Design-First: сначала спецификация, потом код

3. **Почему скрывать документацию в production?**
   - Не раскрывать структуру API злоумышленникам

4. **Какие HTTP коды описывать для CRUD?**
   - 200, 201, 204, 400, 401, 403, 404

## Ссылки

- [Django REST Framework](https://www.django-rest-framework.org/)
- [drf-spectacular](https://drf-spectacular.readthedocs.io/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [JWT.io](https://jwt.io/)
