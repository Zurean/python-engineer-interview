---
type: topic
title: Типизация
tier: 1
impact: 3
effort: 2
status: todo
confidence: 0
tags: [python, tier1]
updated: 2026-10-29
---

# Типизация

> **Зачем на собесе:** на Middle+ ждут, что ты пишешь аннотации и гоняешь mypy/pyright; на Senior — что понимаешь Protocol, дженерики и где проходит граница валидации. Отдельная секция редко, но 2–3 вопроса почти всегда.

## Минимум для собеса

- Аннотации — подсказки, рантайм их не проверяет. Проверяют **mypy** / **pyright**; валидируют в рантайме **Pydantic**.
- Современный синтаксис: `list[int]`, `X | None` вместо `Optional`, `type Alias = ...` и `def f[T](x: T) -> T` (PEP 695, 3.12+).
- **`Protocol`** — структурная типизация («утиная», проверяется по форме). **ABC** — номинальная, требует явного наследования, может давать реализацию по умолчанию. Protocol — когда классы не твои или нужен минимальный контракт; ABC — когда хочешь общую базу и `isinstance`.
- **Дженерики:** `TypeVar`/`def f[T]`, `Generic[T]`, ограничения (`bound=`), `ParamSpec` для декораторов, `TypeVarTuple`.
- **`TypedDict`** — типизированный словарь (для JSON-ответов), **`Literal`**, **`Final`**, **`Annotated`** (используется в FastAPI/Pydantic для метаданных), **`overload`**.
- **Pydantic vs dataclass vs TypedDict:** Pydantic — валидация на границе (HTTP, конфиг, внешние данные); dataclass — внутренние структуры без валидации; TypedDict — форма словаря без класса. Граница валидации — на входе в систему, внутри доверяем типам.
- **Pydantic v2:** `model_validate`, `model_dump`, `Field`, `field_validator`, `model_config`, производительность на Rust-ядре, `BaseSettings` (pydantic-settings).
- Вариантность на пальцах: `list[Dog]` не является `list[Animal]` (инвариантность), `Sequence[Dog]` — является (ковариантность).
- `Any` vs `object`; `cast`, `TYPE_CHECKING`, `# type: ignore[code]`, `reveal_type`.

## Вопросы с собесов

- [ ] В чём разница между `Protocol` и ABC? Когда что? → *структурная vs номинальная*
- [ ] Как типизировать функцию, возвращающую тот же тип, что получила? → *TypeVar / def f[T](x: T) -> T*
- [ ] Pydantic-модель, dataclass, TypedDict — что когда? Где граница валидации? → *на входе системы*
- [ ] Почему `list[Dog]` нельзя передать туда, где ждут `list[Animal]`? → *инвариантность мутабельных контейнеров*
- [ ] Как корректно типизировать декоратор, сохраняющий сигнатуру? → *ParamSpec + TypeVar*
- [ ] Что такое `Annotated` и как его использует FastAPI? → *метаданные: Depends, Query, Field*
- [ ] mypy vs pyright — пользовался? strict-режим? → *рассказать про реальную настройку*
- [ ] Как Pydantic v2 отличается от v1? → *Rust-ядро, model_* API, строгий режим, валидаторы*

## Углубление

- Generic-классы и `Self`, `TypeGuard`/`TypeIs`, `Never`, `assert_never` для исчерпывающего match.
- Дженерик-репозиторий `Repository[T]` в SQLAlchemy-проектах.
- `typing.Protocol` с `@runtime_checkable` и его ограничения.
- Как Pydantic строит core schema, кастомные типы через `__get_pydantic_core_schema__`.
- Gradual typing в legacy-проекте: стратегия внедрения mypy по модулям.

## Ресурсы

- https://typing.python.org/en/latest/ (typing spec и cheat sheets)
- Pydantic docs v2: https://docs.pydantic.dev/latest/
- «Fluent Python», главы 8 и 15

## Связи

[[data-model]] · [[fastapi]] · [[tooling]]

## Мои заметки

