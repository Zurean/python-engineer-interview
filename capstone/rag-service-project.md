---
type: project
title: Сквозной проект — RAG-сервис по документам
status: todo
updated: 2026-10-29
---

# Сквозной проект: RAG-сервис по документам

> **Зачем:** один проект вместо разрозненных курсов. Если был опыт обработки документов через LLM — это его естественное продолжение. Закрывает почти все темы уровней 1–2 и даёт то, чего не даёт теория: *истории для собеса* («а у нас в проекте…»). Это также портфолио на GitHub.

## Что делает сервис

Пользователь загружает документы (PDF/DOCX/MD) → они индексируются → можно задавать вопросы и получать ответы с цитатами источников. Multi-tenant: пользователь видит только свои документы.

## Стек (по вакансиям)

| Компонент | Выбор | Закрывает |
|---|---|---|
| API | FastAPI, async, Pydantic v2 | [fastapi](../30-backend-design/fastapi.md), [typing](../10-python-core/typing.md), [api-design](../30-backend-design/api-design.md) |
| БД | Postgres + pgvector, SQLAlchemy 2.0 async, Alembic | [_databases](../20-databases/_databases.md), [rag](../70-llm-engineering/rag.md) |
| Очередь индексации | Kafka (aiokafka) или Redpanda; retry-topic + DLQ | [kafka](../50-messaging-caching/kafka.md), [resilience-patterns](../30-backend-design/resilience-patterns.md) |
| Кэш и лимиты | Redis: кэш ответов, rate limiting, идемпотентность | [redis](../50-messaging-caching/redis.md) |
| Фоновые задачи | Taskiq/arq для переиндексации и отчётов | [task-queues](../50-messaging-caching/task-queues.md) |
| LLM-слой | SDK напрямую: эмбеддинги, structured output, tool calling для «поиск → ответ» | [rag](../70-llm-engineering/rag.md), [tool-calling-structured-output](../70-llm-engineering/tool-calling-structured-output.md) |
| Evals | датасет 50+ вопросов, recall@k + LLM-judge, запуск в CI | [evals-cost-latency](../70-llm-engineering/evals-cost-latency.md) |
| Тесты | pytest, testcontainers (Postgres, Redis, Kafka) | [_testing](../60-testing/_testing.md) |
| Контейнеры | Dockerfile multi-stage, docker-compose для dev | [docker](../40-infra/docker.md) |
| Оркестрация | kind/k3s: Deployment, Service, Ingress, probes, Job для миграций | [kubernetes](../40-infra/kubernetes.md) |
| CI/CD | GitHub Actions: lint → types → tests → build → deploy в kind | [ci-cd](../40-infra/ci-cd.md) |
| Observability | structlog JSON, Prometheus + Grafana (RED + токены/стоимость), OTel-трейсы, Sentry | [observability](../40-infra/observability.md) |
| Auth | JWT access/refresh, owner-check на документах | [auth-security](../30-backend-design/auth-security.md) |

## Этапы (каждый — рабочий инкремент, 1–2 недели)

- [ ] **0. Скелет.** uv, ruff, mypy, pre-commit, src-layout, FastAPI «hello», Dockerfile, compose с Postgres/Redis, CI с линтом и тестами. → [tooling](../10-python-core/tooling.md), [docker](../40-infra/docker.md), [ci-cd](../40-infra/ci-cd.md)
- [ ] **1. Документы.** Загрузка файла (presigned-подобный поток или прямой upload), таблицы `documents`/`chunks`, миграции, JWT-auth, owner-check, интеграционные тесты на реальном Postgres. → [sqlalchemy-alembic](../20-databases/sqlalchemy-alembic.md), [auth-security](../30-backend-design/auth-security.md), [integration-testing](../60-testing/integration-testing.md)
- [ ] **2. Индексация асинхронно.** Событие «документ загружен» → Kafka → воркер: парсинг, чанкинг, эмбеддинги → pgvector. Outbox для события, идемпотентность по hash, retry-topic, DLQ. → [kafka](../50-messaging-caching/kafka.md), [resilience-patterns](../30-backend-design/resilience-patterns.md)
- [ ] **3. Вопрос-ответ.** Гибридный поиск (вектор + tsvector), rerank, structured output с цитатами, стриминг ответа (SSE). Таймауты/ретраи/фолбэк на вызовы модели. → [rag](../70-llm-engineering/rag.md), [tool-calling-structured-output](../70-llm-engineering/tool-calling-structured-output.md)
- [ ] **4. Кэш и лимиты.** Redis: кэш ответов по (tenant, normalized question), rate limiting на пользователя, Idempotency-Key на загрузку. → [redis](../50-messaging-caching/redis.md)
- [ ] **5. Evals.** Датасет вопросов по своим документам, recall@k, faithfulness через judge, отчёт в CI, сравнение двух стратегий чанкинга. → [evals-cost-latency](../70-llm-engineering/evals-cost-latency.md)
- [ ] **6. Observability.** JSON-логи с request_id/trace_id, /metrics (RED + токены + стоимость), OTel → Jaeger, Sentry, дашборд в Grafana, алерт на error rate. → [observability](../40-infra/observability.md)
- [ ] **7. Kubernetes.** Манифесты/Helm, probes, requests/limits, Job для миграций, rolling update без даунтайма, HPA по CPU. → [kubernetes](../40-infra/kubernetes.md), [zero-downtime-migrations](../20-databases/zero-downtime-migrations.md)
- [ ] **8. Нагрузка и профилирование.** locust/k6, найти узкое место, EXPLAIN ANALYZE на поиске, py-spy на воркере, оптимизировать, задокументировать «до/после». → [memory-performance](../10-python-core/memory-performance.md), [query-optimization-n-plus-1](../20-databases/query-optimization-n-plus-1.md)
- [ ] **9. README как кейс для собеса.** Архитектурная схема, решения и трейдоффы (ADR), цифры (латентность, стоимость на запрос, recall), что бы сделал иначе. → [stories](../00-strategy/stories.md), [architecture-layers](../30-backend-design/architecture-layers.md)

## Как превращать проект в ответы на собесе

После каждого этапа — записать в «Мои заметки» 2–3 «истории»: с какой проблемой столкнулся, как решил, что измерил. Это и есть материал для секций «опыт» и «архитектура». Проект без таких заметок — просто код.

## Мои заметки

