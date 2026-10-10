# DormSupport

Документация системы заявок на ремонт в общежитии.

| Файл | Содержание |
|---|---|
| plan.pdf | Требования, API, архитектура и схемы сценариев |
| requirements.md | Требования, роли, правила и критерии приёмки |
| data-model.md | Поля и ограничения 10 сущностей |
| openapi.yaml / openapi.json | OpenAPI 3.0.3, 10 публичных HTTP-операций и 1 внутренняя |
| notifications.md | Публикация, Telegram, повторы и DLQ |
| events.schema.json | Схемы сообщений RabbitMQ |
| diagrams/ | Диаграммы SVG/PNG и исходники Mermaid/DOT |

Для просмотра контрактов импортируйте openapi.yaml или openapi.json в Swagger Editor.

Базы: identity_db, issues_db и notifications_db. Каждый сервис имеет доступ только к своей базе.
