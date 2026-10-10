# Уведомления

## Маршрутизация

| Параметр | Значение |
|---|---|
| Exchange | dorm.issues, direct, durable |
| Routing keys | issue.created, issue.status_changed |
| Очередь | dorm.notifications, durable |
| Properties | delivery_mode=2, content_type=application/json, message_id=eventId |
| Consumer | manual ACK, prefetch=1 |
| DLX | dorm.notifications.dlx, direct, durable |
| DLQ | dorm.notifications.dlq, durable; routing key failed |

## События

```json
{"eventId":"d25ed18f-15d7-4368-bace-c3f156e76c63","eventVersion":1,"type":"issue.created","occurredAt":"2026-10-10T08:00:00Z","data":{"issueId":124,"categoryName":"Электрика","status":"NEW","title":"Не работает свет","descriptionPreview":"Лампа не включается","blockName":"Корпус 1","roomNumber":"203А","locationDetails":"Кухня 3-го этажа"}}
```

```json
{"eventId":"848cf8ea-39bd-4b89-9280-459d840c4f15","eventVersion":1,"type":"issue.status_changed","occurredAt":"2026-10-10T09:00:00Z","data":{"issueId":124,"oldStatus":"NEW","newStatus":"IN_PROGRESS"}}
```

eventId=History.id; Notification.id=eventId, occurredAt=History.changed_at. JSON Schema: events.schema.json. Допустимые переходы: NEW/IN_PROGRESS и IN_PROGRESS/CLOSED.

## Публикация и доставка

1. Issue и History с published_at=NULL коммитятся одной транзакцией в Issues DB.
2. Publisher читает неопубликованные History по changed_at и id раз в 5 секунд. Статусы берутся из конкретной истории, категория, название, описание и место - из неизменяемой заявки. descriptionPreview содержит описание до 500 символов; для длинного текста берутся первые 499 символов и многоточие. Полное описание доступно в списке на сайте.
3. Persistent сообщение публикуется с mandatory=true. Publisher ожидает confirm и отсутствие basic.return; затем устанавливает History.published_at. Топология создаётся до запуска publisher.
4. Consumer проверяет схему. Сохраняет Notification в Notifications DB с id=eventId и полным исходным событием. Вставка повторного id не создаёт новую запись; различающийся payload с тем же id отправляется в DLQ как INVALID_EVENT, существующая Notification не изменяется.
5. Если sent_at или failed_at заполнены, повтор подтверждается без запроса Telegram. В остальных случаях consumer увеличивает attempt_count и вызывает Telegram.
6. После успешного ответа HTTP и ok=true сохраняет sent_at в Notifications DB, затем ACK.
7. При недоступной Notifications DB сообщение не подтверждается. До сохранения Notification отправки Telegram нет. Результат записи и запрос Telegram не образуют общую транзакцию.

Notification Service не читает базы Identity и Issues и не вызывает их API. Событие содержит все данные сообщения. Notification.issue_id - логическая ссылка по идентификатору, без внешнего ключа. При падении до первого получения события запись Notification ещё отсутствует; событие остаётся в RabbitMQ.

## Telegram

Одна закрытая группа сотрудников. BOT_TOKEN и CHAT_ID находятся в конфигурации. Бот добавлен в группу и имеет право отправки. Новая заявка содержит номер, категорию, название, краткое описание, корпус, место неисправности и комнату жильца. Сообщение о смене статуса содержит номер и переход. Ссылки и кнопки управления в Telegram отсутствуют. Сотрудник меняет статус в списке заявок в приложении.

POST https://api.telegram.org/bot<BOT_TOKEN>/sendMessage:

```json
{"chat_id":"<CHAT_ID>","text":"Новая заявка №124\nКатегория: Электрика\nМесто: Корпус 1, кухня 3-го этажа\nКомната жильца: 203А\nНе работает свет\nЛампа не включается"}
```

Пример смены статуса:

```text
Заявка №124
Статус: Создана → В работе
```

parse_mode не задаётся. Формат места: blockName и locationDetails; roomNumber выводится отдельно как комната жильца. Статусы отображаются как «Создана», «В работе», «Закрыта». Сообщение строится только из ограниченных полей события; текст помещается в лимит Telegram. Токен скрывается в логах, включая URL.

## Повторы и DLQ

Сеть, 5xx и 429: не более 4 начатых запросов Telegram на Notification. Задержки повторов 10/30/60 секунд; при 429 не меньше parameters.retry_after. Тайм-аут HTTP-запроса 10 секунд. Счётчик хранится в Notification и сохраняется при перезапуске. Один consumer ожидает с непогашенным сообщением, задерживая остальные сообщения очереди.

После лимита, постоянной ошибки 400/403, неверного события или версии consumer публикует копию в DLQ с confirm и проверкой маршрутизации. Для валидного события с совпадающим payload затем сохраняет failed_at и last_error. Некорректное событие не изменяет журнал валидного события. Исходное сообщение подтверждается после сохранения необходимых результатов. При невозможности публикации в DLQ или записи результата ACK не выполняется. 401 требует исправления токена бота и помещается в DLQ как постоянная ошибка.

DLQ содержит {originalEvent, reasonCode, failedAt}; неразбираемое сообщение сохраняется как originalBody. После исправления причины оператор сбрасывает failed_at, attempt_count и last_error соответствующей Notification и возвращает сообщение в основную очередь. Некорректный JSON сохраняется в DLQ без создания Notification.

Сбой после sendMessage до записи sent_at может вызвать повторную отправку. Строгий порядок и exactly-once не гарантируются. Актуальное состояние хранится в Issue. RabbitMQ с одной нодой и постоянным volume не защищает от потери диска.

## Документация интеграций

- [Telegram Bot API](https://core.telegram.org/bots/api#sendmessage)
- [RabbitMQ: confirms и acknowledgements](https://www.rabbitmq.com/docs/confirms)
