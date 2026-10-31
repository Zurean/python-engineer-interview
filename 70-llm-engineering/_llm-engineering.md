---
type: hub
title: LLM engineering
tier: 2
updated: 2026-10-29
---

# LLM engineering — хаб

> **Зачем:** дифференциатор. Обязателен в Магните и 1440, «понимать» — в Альфе и Constructor. Работа с AI-инструментами в разработке — 3 из 7, и о ней спрашивают прямо (Циан). Это то, что отличает от других Middle+ кандидатов и открывает AI-позиции. Если есть реальный кейс с LLM — оформить его как историю, см. [stories](../00-strategy/stories.md).

## Порядок прохождения

1. [ai-assisted-development](ai-assisted-development.md) — спрашивают на обычных backend-собесах; готовится за вечер.
2. [rag](rag.md) — основной паттерн LLM-приложений.
3. [tool-calling-structured-output](tool-calling-structured-output.md) — агенты, структурированный вывод, Pydantic.
4. [evals-cost-latency](evals-cost-latency.md) — что отличает инженера от «промпт-энтузиаста».

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "70-llm-engineering" WHERE type = "topic" SORT impact DESC
```

Без Dataview: [ai-assisted-development](ai-assisted-development.md) · [rag](rag.md) · [tool-calling-structured-output](tool-calling-structured-output.md) · [evals-cost-latency](evals-cost-latency.md)

## Практика

Всё это — [rag-service-project](../capstone/rag-service-project.md). Теория без проекта здесь почти бесполезна: вопросы на собесе — «а как вы оценивали качество?», «а что делали с галлюцинациями?», на которые нужен опыт, а не определения. ML-основы на уровне понятий — [ml-basics](../95-optional/ml-basics.md).
