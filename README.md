# DormSupport documents

В этой папке хранятся ERD, архитектурные схемы, API-контракты, Roadmap и инструкции по запуску.

## Первый этап

- Java 21 + Spring Boot 4.0.8;
- Hibernate / Spring Data JPA подготовлены в backend-сервисах;
- Kafka подключён в Docker Compose;
- PostgreSQL подключён с базами `identity_db`, `issues_db`, `notifications_db`;
- все сервисы используют внутренний порт `8080`;
- наружу опубликован Gateway на `localhost:8080`;
- запуск выполняется командой `docker compose up --build`.

ссылка на Миро: https://miro.com/app/board/uXjVEff9gbU=/?inviteKey=eExYVEVjRC9ITzZBdjE5NTZYUzJERXBtQkk5TFBleWkraXlDWHdqdmNaeGtNZWhmSjRYejFGdWMremRtcWJSY0pKMnFNTHJSWStBcUJDWkhySDdxUUM3R3BUNEpaQ3YzYkdHWnlqbk5aay9DMCt6akg1SjVHTXZMSFhnTlFmWmpQdGo1ZEV3bUdPQWRZUHQzSGl6V2NBPT0hdjE%3D
ссылка на макеты Figma: https://www.figma.com/design/ce6491NYRBatsPe8xLtrJp/Untitled?node-id=0-1&m=dev&t=7H5a8Vx2Pd2mKuKC-1
