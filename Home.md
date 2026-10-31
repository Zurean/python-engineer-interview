---
type: meta
title: Главная
updated: 2026-10-31
---

# 🏠 Подготовка к собесам — главная

> **Цель:** Middle+/Senior Python backend, близко к LLM. Целевой рынок — РФ (300–400 тыс. ₽ на руки), международная удалёнка как опция через 1–2 года.
> **Принцип:** сначала то, что спросят с вероятностью >70 %, потом всё остальное. См. [pareto-plan](00-strategy/pareto-plan.md).

## Навигация

- 📖 [how-to-use](_meta/how-to-use.md) — как пользоваться волтом (5 минут)
- 🧭 [Карта проекта и память агента](_meta/index.md) — задачи, состояние и правила продолжения работы
- 🎯 [pareto-plan](00-strategy/pareto-plan.md) — порядок атаки, что учить прямо сейчас
- 🗓 [sprint](_meta/sprint.md) — текущий фокус на 1–2 недели
- ☑️ [progress](_meta/progress.md) — прогресс по темам (ручной чеклист, работает и на GitHub)
- 📓 [log](_meta/log.md) — журнал: что прошёл, что заметил на собесах
- 📐 [conventions](_meta/conventions.md) — как устроены заметки и как просить их углубить

## Стратегия

- [target-profile](00-strategy/target-profile.md) — целевой профиль и позиционирование
- [vacancy-analysis](00-strategy/vacancy-analysis.md) — откуда взялись уровни: частоты требований по 7 вакансиям
- [interview-formats](00-strategy/interview-formats.md) — из каких секций состоит собес и что в каждой спрашивают
- [stories](00-strategy/stories.md) — истории для behavioral-секции по STAR (каркасы типовых историй)
- [self-presentation](00-strategy/self-presentation.md) — рассказ о себе на 2 минуты

## Темы по уровням

### Уровень 1 — обязательно (требуют 5–7 из 7 вакансий)

| Хаб | Зачем |
|---|---|
| [_python-core](10-python-core/_python-core.md) | Первая секция любого собеса. Asyncio — главный фильтр. |
| [_databases](20-databases/_databases.md) | Требование №1 после самого Python. Индексы, EXPLAIN, транзакции. |
| [_backend-design](30-backend-design/_backend-design.md) | FastAPI, API, отказоустойчивость интеграций. |
| [_testing](60-testing/_testing.md) | pytest + интеграционные тесты. Спрашивают почти всегда, готовиться дёшево. |
| [_infra](40-infra/_infra.md) | Docker обязателен, K8s на уровне пользователя. CI/CD и observability — уровень 2. |
| [_system-design](80-system-design/_system-design.md) | Отдельная секция на Middle+/Senior. Нужен фреймворк, а не знания. |
| [_live-coding](90-live-coding/_live-coding.md) | Live coding и код-ревью. Чистая практика. |

### Уровень 2 — сильно повышает шансы (3–4 из 7)

| Хаб | Зачем |
|---|---|
| [_messaging-caching](50-messaging-caching/_messaging-caching.md) | Kafka, Redis, очереди. Без них «микросервисы» — пустое слово. |
| [_llm-engineering](70-llm-engineering/_llm-engineering.md) | Дифференциатор: RAG, tool calling, evals. Плюс работа с AI-агентами, о которой прямо спрашивают. |
| [ci-cd](40-infra/ci-cd.md), [observability](40-infra/observability.md) | Внутри хаба инфраструктуры. |

### Уровень 3 — только под конкретную цель

- [_optional](95-optional/_optional.md) — облако/Terraform, Go, векторные БД глубже pgvector, ML-основы, английский.

### Сквозной проект

- [rag-service-project](capstone/rag-service-project.md) — один проект, закрывающий уровни 1–2 и дающий материал для рассказа на собесе.

## Что взять следующим (Dataview)

Незакрытые темы, отсортированные: уровень ↑, импакт ↓, усилия ↑. Первая строка — кандидат на ближайшую работу.

```dataview
TABLE WITHOUT ID
  file.link AS "Тема",
  tier AS "Ур.",
  impact AS "Импакт",
  effort AS "Усилия",
  status AS "Статус",
  confidence AS "Увер."
FROM "" 
WHERE type = "topic" AND status != "done"
SORT tier ASC, impact DESC, effort ASC
```

## Сводка прогресса (Dataview)

```dataview
TABLE WITHOUT ID
  tier AS "Уровень",
  length(rows) AS "Всего",
  length(filter(rows, (r) => r.status = "done")) AS "Готово",
  length(filter(rows, (r) => r.status = "in-progress")) AS "В работе"
FROM ""
WHERE type = "topic"
GROUP BY tier
SORT tier ASC
```
