# Модель данных

## User

База: identity_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK |
| login | varchar(64) UNIQUE |
| full_name | varchar(150) |
| role | ADMIN / RESIDENT / REVIEWER |
| password_hash | text NULL до активации |
| block | varchar(50), корпус/блок жильца |
| room_number | varchar(20), комната жильца |
| activated_at | timestamptz NULL |
| created_at | timestamptz |

## ActivationCode

База: identity_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK |
| user_id | uuid FK User |
| code_hash | text |
| expires_at | timestamptz |
| used_at | timestamptz NULL |
| revoked_at | timestamptz NULL |
| created_at | timestamptz |

## IssueCategory

База: issues_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK |
| name | varchar(100) UNIQUE |

## IssueStatus

База: issues_db.

| Поле | Тип и ограничение |
|---|---|
| code | varchar(20) PK |
| name | varchar(50) |

## Issue

База: issues_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK |
| resident_id | uuid; ID-ссылка User.id |
| room_number | varchar(20), снимок |
| category_id | uuid FK IssueCategory |
| status_code | varchar(20) FK IssueStatus |
| title | varchar(150) |
| description | varchar(5000) |
| location_details | varchar(250) |
| created_at | timestamptz |
| updated_at | timestamptz |
| block | varchar(50), снимок блока |

## IssueStatusHistory

База: issues_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK |
| issue_id | uuid FK Issue |
| from_status | varchar(20) FK IssueStatus NULL |
| to_status | varchar(20) FK IssueStatus |
| changed_by | uuid; ID-ссылка User |
| changed_at | timestamptz |
| published_at | timestamptz NULL |

## Notification

База: notifications_db.

| Поле | Тип и ограничение |
|---|---|
| id | uuid PK, eventId |
| issue_id | uuid; ID-ссылка Issue |
| event_type | issue.created / issue.status_changed |
| event_payload | jsonb, полное исходное событие |
| occurred_at | timestamptz, время события |
| received_at | timestamptz, время получения |
| sent_at | timestamptz NULL |
| failed_at | timestamptz NULL |
| attempt_count | integer DEFAULT 0 |
| last_error | text NULL |

## Ограничения

- Для RESIDENT обязательны block и room_number; для ADMIN/REVIEWER эти поля могут отсутствовать.
- ActivationCode: один неиспользованный и неотозванный код на User. Перевыпуск отзывает прежний код; применение и отзыв выполняются под блокировкой User.
- Код нужен только для первой активации. После активации вход выполняется по паролю; код повторно не используется.
- History: UNIQUE(issue_id, to_status); переходы NULL→NEW, NEW→IN_PROGRESS, IN_PROGRESS→CLOSED. Issue и история сохраняются одной локальной транзакцией.
- Publisher изменяет только History.published_at. Начальная история обязательна для каждой заявки.
- Notification.id = eventId и уникален. FK на History и Issue отсутствуют: они расположены в другой БД. Notification появляется после получения события.
- Notification.attempt_count от 0 до 4; sent_at и failed_at не заполняются одновременно. Исходный event_payload сохраняется без изменений.
- Номер комнаты и блок - снимок на момент создания заявки. Чтение списка не обращается к Identity.
- Межбазовые ссылки resident_id, changed_by и Notification.issue_id не являются SQL FK.
- Публичное удаление исторических данных отсутствует.

## Индексы

- User(login) UNIQUE.
- ActivationCode(user_id); частичный UNIQUE(user_id) WHERE used_at IS NULL AND revoked_at IS NULL.
- Issue(resident_id, created_at, id), Issue(status_code, created_at, id).
- History(issue_id, changed_at); History(changed_at) WHERE published_at IS NULL.
- Notification(issue_id, received_at).

## Связи ERD

Сплошные связи на SVG/DOT - связи внутри одной БД. Пунктирные - логические ссылки по ID между сервисами. На связях указана кратность 1:N или 1:1. У события может отсутствовать запись Notification до получения consumer. from_status отсутствует у начальной истории. В Mermaid пунктир и метка reference обозначают логическую связь без SQL FK.

## Временные поля

History.published_at - broker подтвердил событие и маршрут. Notification.received_at - событие записано consumer. sent_at - Telegram подтвердил отправку, результат записан. failed_at - копия подтверждённо помещена в DLQ и результат записан. attempt_count - число начатых запросов Telegram.
