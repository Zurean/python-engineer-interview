---
type: topic
title: Интеграционные тесты и testcontainers
tier: 1
impact: 3
effort: 2
status: todo
confidence: 0
tags: [testing, tier1]
updated: 2026-10-29
---

# Интеграционные тесты, testcontainers

> **Зачем на собесе:** вакансии прямо просят интеграционные тесты. Вопрос «как поднимаешь базу в тестах» — обязательный. Хороший ответ: реальный Postgres в контейнере, транзакция на тест, Kafka/Redis тоже в контейнерах.

## Минимум для собеса

- **Что это:** тесты, проверяющие работу с реальными зависимостями (БД, брокер, кэш), в отличие от unit (изолированная логика) и e2e (вся система через внешний интерфейс).
- **testcontainers-python:** поднимает Docker-контейнер Postgres/Redis/Kafka на время сессии тестов, даёт URL. Фикстура `scope="session"`. Альтернатива — docker-compose для тестов в CI, но testcontainers переносимее.
- **Изоляция тестов на одной БД:** 1) транзакция на тест с rollback (быстро; осторожно с кодом, который сам коммитит — использовать savepoints/`join_transaction_mode`); 2) truncate после теста; 3) отдельная схема/БД на воркер при `pytest-xdist`. Миграции (Alembic) прогоняются один раз на сессию → заодно тест, что миграции применяются.
- **Почему не SQLite вместо Postgres:** другой диалект (jsonb, массивы, `ON CONFLICT`, типы), другая семантика блокировок — тесты зелёные, прод красный.
- **Тест-данные:** фабрики (`polyfactory` под Pydantic/SQLAlchemy, `factory_boy`), минимально необходимые, читаемые.
- **Внешние HTTP-API:** не поднимать, мокать на уровне транспорта (`respx`) или контрактные тесты/wiremock.
- **Очереди:** Kafka в testcontainers (медленно стартует, 10–20 с) или проверять продюсер через фейк + отдельный тест консюмера на реальном брокере.
- **CI:** Docker-in-Docker или сервис-контейнеры GitHub Actions/GitLab (`services:`); кэш образов; интеграционные тесты отдельной job-ой, unit — быстрой.
- **Скорость:** session-scoped контейнеры, транзакционный rollback, параллелизм; целевой ориентир — весь набор < 5 минут.
- **Smoke/e2e:** несколько сценариев через HTTP на поднятом приложении; не дублировать ими unit-логику.

## Вопросы с собесов

- [ ] Как тестируешь код, работающий с БД? Какую базу поднимаешь? → *реальный Postgres в контейнере, транзакция на тест*
- [ ] Как изолируешь тесты друг от друга на общей БД? → *rollback / truncate / схема на воркер*
- [ ] Почему не SQLite in-memory? → *диалект, типы, блокировки*
- [ ] Как гоняешь интеграционные тесты в CI? → *services/testcontainers, отдельная job*
- [ ] Как тестируешь Kafka-консюмер? → *контейнер или фейковый брокер; идемпотентность на дубликатах*
- [ ] Как тестируешь миграции? → *upgrade head на пустой + downgrade; на копии прод-данных для тяжёлых*
- [ ] Что в твоём проекте относится к unit, что к интеграционным, что к e2e? → *честная пирамида/трапеция*

## Углубление

- Transaction rollback паттерн с async SQLAlchemy: `join_transaction_mode="create_savepoint"`.
- Тестирование с репликой/лагом, тесты на блокировки и deadlock (два параллельных воркера).
- Тестовые двойники для LLM-вызовов: записанные ответы (VCR-подход), детерминированные фейки; evals как отдельный вид тестов ([evals-cost-latency](../70-llm-engineering/evals-cost-latency.md)).
- Consumer-driven contracts (pact) между сервисами.
- Load/perf-тесты (locust, k6) как часть CI на ветке релиза.

## Ресурсы

- https://testcontainers-python.readthedocs.io/
- SQLAlchemy: «Joining a Session into an External Transaction» (docs, раздел для тестов)
- Martin Fowler, «Test Pyramid» / «Integration Test»

## Связи

[pytest](pytest.md) · [sqlalchemy-alembic](../20-databases/sqlalchemy-alembic.md) · [docker](../40-infra/docker.md) · [ci-cd](../40-infra/ci-cd.md) · [kafka](../50-messaging-caching/kafka.md)

## Мои заметки

