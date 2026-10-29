---
type: topic
title: Go как второй язык
tier: 3
impact: 2
effort: 4
status: todo
confidence: 0
tags: [lang, tier3]
updated: 2026-10-29
---

# Go как второй язык

> **Зачем на собесе:** Бюро 1440 — обязателен, Constructor — плюс. В РФ-бэкенде Go — частый сосед Python (высоконагруженные сервисы), и «понимаю, когда взять Go» звучит сеньорно даже без глубокого знания.

## Минимум (стаб)

- Уровень «читаю и пишу простой сервис»: синтаксис, структуры, интерфейсы (неявные), ошибки как значения, goroutines/channels/`select`, `context`, `sync`, стандартный `net/http`, модули.
- Уметь объяснить трейдофф Python vs Go: производительность и параллелизм vs скорость разработки и экосистема (ML/LLM).
- Один маленький проект: HTTP-сервис с Postgres и тестами (A Tour of Go + «Let's Go» или gobyexample).

## Вопросы с собесов

- [ ] Когда бы выбрал Go вместо Python для сервиса? → *CPU/латентность/параллелизм, простые бинарники*
- [ ] Как устроена конкурентность в Go и чем отличается от asyncio? → *goroutines (M:N), каналы, вытесняющий планировщик*

## Ресурсы

- https://go.dev/tour/ , https://gobyexample.com/ , Alex Edwards «Let's Go»

## Связи

[[asyncio]] · [[gil-threads-processes]] · [[target-profile]]

## Мои заметки

