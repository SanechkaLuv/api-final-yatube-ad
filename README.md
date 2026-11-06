# API для Yatube

Учебный REST API для социальной сети Yatube на Django REST Framework. Через API можно читать и публиковать записи, комментировать публикации, просматривать сообщества и подписываться на авторов. Для изменения данных используется JWT-аутентификация.

## Возможности

- Публикации: GET, POST, PUT, PATCH и DELETE; посты доступны всем для чтения, редактировать и удалять их может только автор.
- Комментарии: создание, просмотр, изменение и удаление внутри конкретной публикации. Изменять и удалять комментарии может только их автор.
- Сообщества: получение списка и подробной информации, без записи через API.
- Подписки: просмотр своих подписок, поиск по имени автора и создание подписки. Самоподписка и повторная подписка запрещены.
- JWT: получение, обновление и проверка токенов.
- Пагинация публикаций по `limit` и `offset`.

## Технологии

Python 3.10, Django 3.2, Django REST Framework 3.12, Simple JWT 4.7, SQLite для локальной разработки.

## Установка

Скачайте проект и перейдите в корневую папку, содержащую `requirements.txt`.

### Windows (PowerShell)

```powershell
py -3.10 -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
cd yatube_api
python manage.py migrate
python manage.py runserver
```

Если PowerShell блокирует активацию виртуального окружения, для текущего окна выполните `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` и повторите команду активации.

### Linux / macOS

```bash
python3.10 -m venv venv
source venv/bin/activate
python -m pip install -r requirements.txt
cd yatube_api
python manage.py migrate
python manage.py runserver
```

При необходимости создайте тестового пользователя командой `python manage.py createsuperuser`. После запуска документация доступна по адресу <http://127.0.0.1:8000/redoc/>.

## Эндпоинты API

| Метод | Адрес | Назначение |
|---|---|---|
| GET, POST | `/api/v1/posts/` | Список и создание публикаций |
| GET, PUT, PATCH, DELETE | `/api/v1/posts/{id}/` | Конкретная публикация |
| GET | `/api/v1/groups/` | Все сообщества |
| GET | `/api/v1/groups/{id}/` | Конкретное сообщество |
| GET, POST | `/api/v1/posts/{post_id}/comments/` | Комментарии публикации |
| GET, PUT, PATCH, DELETE | `/api/v1/posts/{post_id}/comments/{id}/` | Конкретный комментарий |
| GET, POST | `/api/v1/follow/` | Подписки пользователя |
| POST | `/api/v1/jwt/create/` | Получить access и refresh |
| POST | `/api/v1/jwt/refresh/` | Обновить access |
| POST | `/api/v1/jwt/verify/` | Проверить токен |

Анонимные пользователи могут выполнять GET-запросы к публикациям, комментариям и сообществам. Для других операций необходим JWT в заголовке `Authorization: Bearer <access-токен>`. Подписки недоступны анонимным пользователям.

## Примеры запросов

Получение JWT (пользователь должен быть создан заранее):

```http
POST /api/v1/jwt/create/
Content-Type: application/json

{"username": "test_user", "password": "your_password"}
```

Создание публикации:

```http
POST /api/v1/posts/
Authorization: Bearer <access-токен>
Content-Type: application/json

{"text": "Первая запись", "group": 1}
```

Получение публикаций с пагинацией:

```http
GET /api/v1/posts/?limit=10&offset=0
```

Создание комментария к публикации 1:

```http
POST /api/v1/posts/1/comments/
Authorization: Bearer <access-токен>
Content-Type: application/json

{"text": "Отличная запись!"}
```

Подписка на автора и поиск подписок:

```http
POST /api/v1/follow/
Authorization: Bearer <access-токен>
Content-Type: application/json

{"following": "another_user"}
```

```http
GET /api/v1/follow/?search=another
Authorization: Bearer <access-токен>
```

При успешном создании объекта API возвращает HTTP 201, при удалении — 204, при недостаточных правах — 401 либо 403, при обращении к отсутствующему объекту — 404.

## Тестирование

Из корневой директории, содержащей `pytest.ini`, выполните:

```bash
python -m pytest -v
```

Исходные тесты находятся в каталоге `tests/`; коллекция запросов Postman — в `postman_collection/`.
