---
type: hub
title: Уровень 3 — под конкретную цель
tier: 3
updated: 2026-10-29
---

# Уровень 3 — только под конкретную цель

> **Зачем:** каждая тема здесь встречается в 0–1 вакансии из 7. Брать только когда выбрана конкретная цель (международная удалёнка → [english](english.md) + [cloud-terraform](cloud-terraform.md); Бюро 1440 → [go-lang](go-lang.md)) или когда уровни 1–2 закрыты. Заметки здесь — короткие стабы с ориентирами; при необходимости просить развернуть.

## Подтемы

| Тема | Когда нужна |
|---|---|
| [cloud-terraform](cloud-terraform.md) | Международные вакансии; РФ — Yandex Cloud/VK Cloud, редко спрашивают |
| [english](english.md) | Международные вакансии (Constructor — C1) |
| [go-lang](go-lang.md) | 1440 (обязателен), Constructor (плюс); частый «второй язык» в РФ-бэкенде |
| [vector-dbs](vector-dbs.md) | AI-позиции при масштабе больше pgvector |
| [ml-basics](ml-basics.md) | AI-позиции: уровень «понимать», не «обучать» |

```dataview
TABLE WITHOUT ID file.link AS "Тема", impact AS "Импакт", effort AS "Усилия", status AS "Статус"
FROM "95-optional" WHERE type = "topic" SORT impact DESC
```
