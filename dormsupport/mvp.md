# DormSupport MVP

## Цель

Сделать минимальную систему заявок на ремонт в общежитии.

Администратор создаёт аккаунты пользователей и передаёт им первоначальные логин и пароль. Жилец входит в систему и создаёт заявку. Администратор и ответственный сотрудник видят заявки и меняют их статусы.

## Архитектура

```text
Browser :8080
    |
    v
Gateway :8080
    |-- /api/v1/auth/**       -> auth-service:8080
    |-- /api/v1/issues/**     -> issues-service:8080
    |-- /api/v1/categories    -> issues-service:8080
    |-- /                   -> frontend:80
    |
    +--> auth-service --> identity_db
    +--> issues-service --> issues_db
                    |
                    +--> Kafka: issue-events
                              |
                              v
                       notifications-service --> notifications_db
```

Внешний порт только один:

```text
http://localhost:8080
```

Frontend, Swagger и API доступны через Gateway.

## Роли

### RESIDENT

- войти в систему;
- создать заявку;
- просматривать собственные заявки.

### ADMIN

- создавать аккаунты `RESIDENT` и `REVIEWER`;
- войти в систему;
- просматривать все заявки;
- фильтровать заявки по статусу;
- менять статус заявки.

### REVIEWER

- войти в систему;
- просматривать все заявки;
- фильтровать заявки по статусу;
- менять статус заявки.

## Авторизация

На первом этапе не используются JWT и activation code. Самостоятельной регистрации нет: аккаунты создаёт `ADMIN`.

`POST /api/v1/auth/login` создаёт серверную сессию и возвращает `sessionId`. В следующих запросах frontend передаёт:

```text
X-Session-Id: <session-id>
```

## Идентификаторы

Идентификаторы пользователей, корпусов и связанные с ними ссылки используют
`UUID`. Идентификаторы заявок и категорий пока остаются числовыми (`bigint`),
поскольку это отдельные сущности Issues Service.

## MVP API

```http
POST  /api/v1/admin/users
POST  /api/v1/auth/login
GET   /api/v1/auth/me
GET   /api/v1/categories
POST  /api/v1/issues
GET   /api/v1/issues
GET   /api/v1/issues/{issueId}
PATCH /api/v1/issues/{issueId}/status
```

Заявка создаётся с полями:

```json
{
  "title": "Не работает лампочка",
  "description": "Лампа не включается",
  "categoryId": 1,
  "locationDetails": "Коридор, 3 этаж"
}
```

Автор, блок, комната, дата и начальный статус определяются сервером.

## Жизненный цикл заявки

```text
NEW -> IN_PROGRESS -> CLOSED
```

Другие переходы в MVP запрещены.

## Kafka

`issues-service` публикует события:

```text
issue.created
issue.status_changed
```

`notifications-service` получает события из Kafka и сохраняет их в `notifications_db`. Telegram, RabbitMQ, retry и DLQ в MVP не входят.

## Swagger

Публичная документация Gateway:

```text
http://localhost:8080/swagger-ui.html
```

OpenAPI-файлы сервисов находятся в соответствующих репозиториях в `src/main/resources/static/openapi.yaml`.
