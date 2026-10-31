---
type: topic
title: Модель данных Python
tier: 1
impact: 4
effort: 2
status: todo
confidence: 0
tags: [python, tier1]
updated: 2026-10-29
---

# Модель данных Python

> **Зачем на собесе:** классика технического скрининга. Вопросы предсказуемы: мутабельные дефолты, `__slots__`, дескрипторы, MRO, `is` vs `==`. Готовится за 2 вечера, спрашивают в 100 % случаев.

## Минимум для собеса

- **Всё — объект.** У каждого объекта есть identity (`id`), тип и значение. `is` сравнивает identity, `==` вызывает `__eq__`.
- **Мутабельные / иммутабельные:** `list, dict, set` — мутабельные; `int, str, tuple, frozenset` — нет. Мутабельный дефолт аргумента `def f(x=[])` создаётся один раз при определении функции → накапливает состояние. Фикс: `x=None` + создание внутри.
- **Хеширование:** ключи dict и элементы set должны быть хешируемыми; переопределил `__eq__` — переопредели `__hash__` (иначе он становится `None`).
- **Dunder-методы:** `__init__` vs `__new__` (создание объекта), `__repr__` vs `__str__`, `__eq__`/`__hash__`, `__len__`/`__iter__`/`__getitem__` (протоколы контейнеров), `__enter__`/`__exit__`, `__call__`.
- **`__getattr__` vs `__getattribute__`:** первый вызывается только когда атрибут *не найден* обычным путём; второй — на *каждый* доступ (легко уйти в рекурсию, использовать `object.__getattribute__`).
- **Дескрипторы:** объект с `__get__`/`__set__`/`__delete__` на уровне класса. На них построены `property`, методы (функции — дескрипторы, отсюда `self`), `classmethod`, `staticmethod`. Data-дескриптор (есть `__set__`) имеет приоритет над instance `__dict__`.
- **MRO:** C3-линеаризация; `Class.__mro__`. `super()` идёт по MRO *экземпляра*, а не к «родителю» — поэтому работают миксины и cooperative multiple inheritance.
- **`__slots__`:** убирает `__dict__` у экземпляра → меньше памяти, быстрее доступ, нельзя добавлять произвольные атрибуты.
- **dataclasses:** генерируют `__init__`, `__repr__`, `__eq__`; `frozen=True`, `slots=True`, `field(default_factory=list)`, `kw_only=True`. Отличие от `NamedTuple`, `TypedDict`, Pydantic — см. [typing](typing.md).
- **Копирование:** `copy.copy` (поверхностная) vs `copy.deepcopy`; присваивание копию не делает.
- **Области видимости:** LEGB, `global`, `nonlocal`; замыкания захватывают переменную, а не значение (ловушка с лямбдами в цикле).

## Вопросы с собесов

- [ ] Что выведет `def f(x=[]): x.append(1); return x` при двух вызовах? Почему? Как исправить? → *[1], [1, 1]; дефолт вычисляется один раз; x=None*
- [ ] Чем `__getattr__` отличается от `__getattribute__`? → *только при отсутствии vs на каждый доступ*
- [ ] Что такое дескриптор и где он используется в стандартной библиотеке? → *property, методы, classmethod, staticmethod*
- [ ] Как работает MRO при множественном наследовании и что на самом деле делает `super()`? → *C3, следующий по MRO экземпляра*
- [ ] `is` vs `==`? Почему `a = 256; b = 256; a is b` — True, а для 257 — зависит? → *кэш малых int, деталь реализации CPython*
- [ ] Зачем `__slots__`, какие минусы? → *память/скорость; нет __dict__, проблемы с множественным наследованием*
- [ ] Переопределил `__eq__`, объект перестал класться в set. Почему? → *__hash__ стал None*
- [ ] `__new__` vs `__init__`? Когда нужен `__new__`? → *создание vs инициализация; иммутабельные типы, синглтоны*
- [ ] Чем dataclass отличается от обычного класса и от NamedTuple? → *генерация методов; NamedTuple — иммутабельный кортеж*
- [ ] Что такое замыкание и почему `[lambda: i for i in range(3)]` все возвращают 2? → *late binding; фикс через аргумент по умолчанию*
- [ ] Как работает `*args, **kwargs`, keyword-only и positional-only параметры (`/`, `*`)?

## Углубление

- Метаклассы: `type` как метакласс, `__init_subclass__` как замена в 90 % случаев, `__class_getitem__`.
- Протоколы итерации в деталях: `__iter__` vs `__getitem__`-фолбэк, `iter(callable, sentinel)`.
- Как реализованы dict (compact dict, порядок вставки с 3.7) и set; почему хеш-коллизии не страшны.
- Интернирование строк, кэширование малых int — почему на это нельзя полагаться.
- `__set_name__`, дескрипторы для валидации полей (как в Pydantic/attrs).
- Weak references, `__del__` и почему на него нельзя рассчитывать.

> Запросить углубление: «углуби *Углубление* в `data-model` про метаклассы / внутренности dict».

## Ресурсы

- Python Data Model: https://docs.python.org/3/reference/datamodel.html
- Luciano Ramalho, «Fluent Python» (2nd ed.) — главы 1, 11–13, 23–24
- Descriptor HowTo Guide: https://docs.python.org/3/howto/descriptor.html

## Связи

[generators-context-managers-decorators](generators-context-managers-decorators.md) · [typing](typing.md) · [memory-performance](memory-performance.md) · [python-live-coding-tasks](../90-live-coding/python-live-coding-tasks.md)

## Мои заметки

