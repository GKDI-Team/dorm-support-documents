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
