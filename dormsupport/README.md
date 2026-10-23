# DormSupport

Документация системы заявок на ремонт в общежитии.

## Текущий MVP

Актуальный план MVP находится в `mvp.md`. В MVP используются простая регистрация по логину и паролю, роли `RESIDENT` и `ADMIN`, Kafka для событий и единая внешняя точка входа Gateway на `http://localhost:8080`.

Swagger-контракты сервисов хранятся рядом с их кодом:

- `auth-service/src/main/resources/static/openapi.yaml`
- `gateway-service/src/main/resources/static/openapi.yaml`
- `issues-service/src/main/resources/static/openapi.yaml`
- `notifications-service/src/main/resources/static/openapi.yaml`

`requirements.md`, `openapi.yaml`, `events.schema.json` и `notifications.md` ниже описывают целевую расширенную версию системы и должны рассматриваться как target architecture, пока не будут синхронизированы с MVP.

| Файл | Содержание |
|---|---|
| plan.pdf | Требования, API, архитектура и схемы сценариев |
| requirements.md | Требования, роли, правила и критерии приёмки |
| data-model.md | Поля и ограничения 10 сущностей |
| openapi.yaml / openapi.json | OpenAPI 3.0.3, 10 публичных HTTP-операций и 1 внутренняя |
| mvp.md | Актуальный объём первого этапа |
| notifications.md | Целевая публикация и доставка уведомлений |
| events.schema.json | Целевая схема событий |
| diagrams/ | Диаграммы SVG/PNG и исходники Mermaid/DOT |

Для просмотра контрактов импортируйте openapi.yaml или openapi.json в Swagger Editor.

Базы: identity_db, issues_db и notifications_db. Каждый сервис имеет доступ только к своей базе.
