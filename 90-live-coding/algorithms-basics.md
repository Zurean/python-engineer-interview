---
type: topic
title: Алгоритмическая база
tier: 1
impact: 3
effort: 4
status: todo
confidence: 0
tags: [practice, algorithms, tier1]
updated: 2026-10-29
---

# Алгоритмическая база для live coding

> **Зачем на собесе:** в банках, Яндексе, Озоне, Авито есть алгоритмическая секция (обычно 2 задачи easy/medium за час). В продуктовых компаниях чаще прикладная задача. Цель — не LeetCode-гринд, а уверенность в базовых структурах и сложности, чтобы не слиться на простом.

## Минимум для собеса

- **Сложность:** O(1), O(log n), O(n), O(n log n), O(n²); уметь назвать для своего кода. Память тоже.
- **Структуры в Python и их стоимость:** `list` (индекс O(1), вставка в начало O(n)), `dict`/`set` (O(1) средн.), `deque` (O(1) с обоих концов), `heapq` (min-heap, O(log n)), `sorted`/`sort` (Timsort, O(n log n), стабильная, `key=`), `bisect`, `collections.Counter/defaultdict`.
- **Приёмы, покрывающие 80 % задач easy/medium:** хеш-таблица для поиска пар/дубликатов; два указателя; скользящее окно; префиксные суммы; стек для скобок/монотонных задач; сортировка + жадность; BFS/DFS по графу/сетке; бинарный поиск по ответу; простая динамика (одномерная); куча для top-k.
- **Строки:** `split/join`, срезы, `Counter`, anagram/palindrome-паттерны.
- **Связный список, дерево** — знать обход, реверс списка, глубину дерева; в Python встречаются реже, но бывают.
- **Как решать:** пример → brute force → узкое место → структура данных, которая его убирает → код → прогон по примеру → edge cases.

## План практики (минимально достаточный)

Решать 3–4 задачи в неделю, 30–40 задач суммарно, по темам (LeetCode / neetcode.io roadmap «Neetcode 150», только easy и часть medium):
- [ ] Arrays & Hashing: Two Sum, Group Anagrams, Top K Frequent, Product Except Self
- [ ] Two Pointers: Valid Palindrome, Two Sum II, Container With Most Water
- [ ] Sliding Window: Best Time to Buy/Sell, Longest Substring Without Repeating, Min Window (medium+)
- [ ] Stack: Valid Parentheses, Min Stack, Daily Temperatures
- [ ] Binary Search: Binary Search, Search Rotated Array, Koko Eating Bananas
- [ ] Linked List: Reverse, Merge Two Sorted, Detect Cycle, LRU Cache
- [ ] Trees: Max Depth, Invert, Level Order (BFS), Validate BST
- [ ] Graphs: Number of Islands, Clone Graph, Course Schedule (topological sort)
- [ ] Heap: Kth Largest, K Closest Points, Task Scheduler
- [ ] 1D DP: Climbing Stairs, House Robber, Coin Change
- [ ] Intervals: Merge Intervals, Meeting Rooms
- [ ] Прикладные: парсинг лога потоково, merge k отсортированных файлов, top-N слов в тексте, интервалы занятости, валидация и нормализация данных

## Вопросы с собесов

- [ ] Сложность `in` для list/set/dict, `list.insert(0, x)`, `sorted`? → *O(n)/O(1)/O(1), O(n), O(n log n)*
- [ ] Как найти top-k элементов в потоке? → *heap размера k, O(n log k)*
- [ ] Как проверить, есть ли цикл в графе зависимостей? → *DFS с цветами / Kahn's topological sort*
- [ ] Как объединить k отсортированных последовательностей? → *heapq.merge / heap*
- [ ] Почему sort в Python стабильный и зачем это нужно? → *Timsort; сортировка по нескольким ключам*

## Углубление

- Графы глубже: Dijkstra, Union-Find, двухмерная DP, backtracking — только если целишься в Яндекс/алгоритмические секции Senior.
- Разбор задач на Python-специфику: генераторы в алгоритмах, `itertools`, память.

## Ресурсы

- https://neetcode.io/roadmap (Neetcode 150, фильтр easy/medium)
- Книга «Грокаем алгоритмы» (Бхаргава) — если база совсем слабая, 2 дня
- https://leetcode.com/ — list «Top Interview 150»

## Связи

[python-live-coding-tasks](python-live-coding-tasks.md) · [data-model](../10-python-core/data-model.md) · [memory-performance](../10-python-core/memory-performance.md)

## Мои заметки

