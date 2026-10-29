---
type: strategy
title: Анализ требований вакансий
updated: 2026-10-29
---

# Анализ требований: 7 вакансий (октябрь 2026)

Источник уровней важности во всём волте. Средняя вакансия — Python-бэкендер Middle+/Senior с опытом ~4 года (в вакансиях: 3, 3, 3, 4, 5, 5, 6 лет). Стек: async FastAPI, глубокий PostgreSQL, Kafka, Redis, Docker/K8s, тесты, мониторинг. Сверху — умение встраивать LLM.

## Частота требований

| Навык | Из 7 | Где | Заметки |
|---|---|---|---|
| Python | 7 | все | [[_python-core]] |
| PostgreSQL / SQL | 6 | все, кроме Constructor | [[_databases]] |
| Проектирование: микросервисы, архитектура | 6 | все, кроме 1440 | [[_system-design]], [[architecture-layers]] |
| Async: FastAPI, aiohttp, asyncio | 5 | Альфа, X5, Циан, Банк, 1440 | [[asyncio]], [[fastapi]] |
| Kubernetes | 5 | Альфа, X5, Циан, Constructor, 1440 | [[kubernetes]] |
| Docker | 5 | Альфа, Циан, Банк, 1440, X5 | [[docker]] |
| Kafka / RabbitMQ | 4 | Альфа, X5, Циан, Банк | [[kafka]] |
| SQLAlchemy / Alembic | 4 | Альфа, X5, Циан, Банк | [[sqlalchemy-alembic]] |
| Тесты (unit + интеграционные) | 4 | X5, Циан, Банк, 1440 | [[_testing]] |
| Observability | 4 | Циан, Банк, Constructor, Магнит | [[observability]] |
| AI-инструменты в разработке | 3 | Циан (спрашивают на собесе), Банк (выдают Claude Code), Constructor | [[ai-assisted-development]] |
| Redis | 3 | Альфа, Банк, 1440 | [[redis]] |
| CI/CD | 3 | X5, Циан (Jenkins), Банк | [[ci-cd]] |
| Оптимизация БД и производительности | 3 | X5, Циан, 1440 | [[query-optimization-n-plus-1]], [[memory-performance]] |
| LLM: RAG, tool calling, structured output | 2 + 2 рядом | Магнит, 1440; Альфа и Constructor — «понимать ML» | [[_llm-engineering]] |
| Evals | 2 | Магнит, 1440 | [[evals-cost-latency]] |
| Go / Rust | 2 | 1440 (Go обязателен), Constructor (плюс) | [[go-lang]] |
| Облако, Terraform, векторные БД, английский | 1 | Constructor | [[_optional]] |
| Фронт, классический MLOps | 0 | — | — |

## Уровни

**Уровень 1 — обязательно (5–7 из 7):** asyncio + FastAPI; PostgreSQL (индексы, EXPLAIN, транзакции, SQLAlchemy 2.0, Alembic); проектирование API и микросервисов, обработка отказов; Docker + K8s на уровне пользователя; pytest + интеграционные тесты; работа с AI-агентом и ревью его кода.

**Уровень 2 — сильно повышает шансы (3–4 из 7):** Kafka; Redis; CI/CD; observability; профилирование; LLM-слой (RAG, tool calling, structured output, evals, стоимость/задержка).

**Уровень 3 — под конкретную цель:** облако и Terraform, английский (международка); Go (1440); векторные БД глубже pgvector; основы ML.

## Вакансии

- Магнит Tech — Разработчик AI / LLM
- Альфа-Банк — Backend Python Developer
- X5 Tech — Senior Python Developer
- Циан — Middle+ Python Developer
- Банк (скрыт) — Product Engineer (Python), команда аналитики
- Constructor.io — Senior Backend Engineer, ML Infrastructure
- Бюро 1440 — LLM Engineer

## Как обновлять

Когда появятся новые вакансии — добавить строку в таблицу и пересчитать. Если навык перешёл порог 5/7 — поднять его заметке `tier`.
