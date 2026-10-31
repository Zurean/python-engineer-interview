---
type: hub
title: Live coding и практика
tier: 1
updated: 2026-10-29
---

# Live coding и практика — хаб

> **Зачем:** на большинстве собесов есть секция, где надо писать код при интервьюере. В банках и Яндексе — алгоритмы (LeetCode easy/medium), в продуктовых — прикладная задача на Python (парсинг, асинхронная обработка, рефакторинг) или ревью кода. Это навык, он тренируется только практикой.

## Порядок прохождения

1. [python-live-coding-tasks](python-live-coding-tasks.md) — все скрининговые вопросы и задачи «напиши на доске» одним списком. Прогнать без подглядывания.
2. [code-review-exercise](code-review-exercise.md) — упражнение на ревью с типовыми багами. Спрашивают в Циане и везде, где есть AI-инструменты.
3. [algorithms-basics](algorithms-basics.md) — минимальная алгоритмическая база: структуры данных, сложность, 30–50 типовых задач.

## Подтемы

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус", confidence AS "Увер."
FROM "90-live-coding" WHERE type = "topic" SORT impact DESC
```

Без Dataview: [python-live-coding-tasks](python-live-coding-tasks.md) · [code-review-exercise](code-review-exercise.md) · [algorithms-basics](algorithms-basics.md)

## Правила поведения на live coding

- Проговорить понимание задачи и примеры входа/выхода до кода. Уточнить граничные случаи (пусто, один элемент, дубликаты, большие объёмы).
- Сначала простое рабочее решение, потом оптимизация — вслух: «это O(n²), могу улучшить до O(n) через set».
- Писать читаемо: имена, маленькие функции, type hints. Не молчать.
- Тестировать руками: прогнать пример по коду, назвать edge cases.
- Если застрял — сказать, что думаешь, попросить подсказку. Это нормально.
