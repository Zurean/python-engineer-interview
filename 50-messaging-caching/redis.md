---
type: topic
title: Redis
tier: 2
impact: 4
effort: 2
status: todo
confidence: 0
tags: [caching, tier2]
updated: 2026-10-29
---

# Redis: кэш, локи, rate limiting

> **Зачем на собесе:** 3 из 7 вакансий напрямую; плюс «как кэшировать», «как сделать распределённый лок», «rate limiter» — в каждом системном дизайне. Тема компактная, импакт высокий.

## Минимум для собеса

- **Что это:** in-memory key-value с структурами данных, однопоточная обработка команд (атомарность каждой команды), опциональная персистентность (RDB-снапшоты, AOF). Latency < 1 мс.
- **Структуры и применение:** String (кэш, счётчики `INCR`), Hash (объект), List (очередь, `LPUSH/BRPOP`), Set (уникальные, теги), Sorted Set (рейтинги, sliding window, отложенные задачи по score=time), Streams (лог событий, consumer groups — лёгкая Kafka), HyperLogLog (уникальные приблизительно), Bitmap, Geo. Pub/Sub (fire-and-forget, без persistence).
- **Кэширование:** cache-aside — главный паттерн; TTL всегда; ключи с префиксом и версией (`user:v2:{id}`); инвалидация по событию записи (`DEL`) или TTL; **cache stampede** — лок на пересчёт или early/probabilistic refresh; **hot keys** — локальный кэш поверх; **не кэшировать ошибки** надолго; размер значений. Что кэшировать: дорогие и часто читаемые, редко меняющиеся.
- **Распределённый лок:** `SET key value NX PX ttl` + удаление только своим value через Lua (проверка владельца). TTL обязателен (держатель может умереть). Redlock — спорный (Kleppmann), для критичных вещей — лок в БД/advisory lock или fencing token.
- **Rate limiting:** fixed window (`INCR` + `EXPIRE`), sliding window log (sorted set), token bucket через Lua-скрипт (атомарно). Fail-open или fail-closed при недоступности Redis — осознанный выбор.
- **Атомарность:** `MULTI/EXEC` (транзакция без отката), `WATCH` (optimistic), **Lua-скрипты** (атомарно, `EVALSHA`), pipelining (меньше round-trip).
- **Память:** `maxmemory` + политика вытеснения (`allkeys-lru`, `volatile-ttl`); большие ключи и `KEYS *` в проде — нельзя (`SCAN`).
- **Отказоустойчивость:** Sentinel (failover), Cluster (шардирование по hash slots, ограничения на multi-key операции — hash tags `{user:1}`). Persistence: RDB vs AOF, что теряем при падении.
- **Python:** `redis` (`redis.asyncio`), пул соединений, таймауты (кэш не должен тормозить при деградации Redis — маленький таймаут + fallback в БД).
- **Сессии, idempotency keys, дедупликация, счётчики, лидерборды, feature flags** — типовые применения помимо кэша.

## Вопросы с собесов

- [ ] Как кэшируешь и как инвалидируешь? → *cache-aside, TTL + DEL on write, версии ключей*
- [ ] Что такое cache stampede и как бороться? → *лок/single-flight, early refresh, jitter TTL*
- [ ] Как сделать распределённый лок на Redis и в чём его слабости? → *SET NX PX, Lua-release, TTL; не для критичной консистентности*
- [ ] Реализуй rate limiter на Redis. → *token bucket в Lua / sliding window в zset*
- [ ] Redis однопоточный — почему быстрый? Что его тормозит? → *память, epoll, нет локов; большие ключи, KEYS, Lua долгий*
- [ ] Что будет с данными при падении Redis? → *зависит от RDB/AOF; кэш — переживём, state — нужен AOF/реплика*
- [ ] Чем Redis Streams отличается от Kafka? → *проще, меньше гарантий/масштаба; подходит для небольших очередей*
- [ ] Как Redis Cluster распределяет ключи и что с multi-key операциями? → *hash slots; hash tags*
- [ ] Что делать, если Redis недоступен? → *таймаут, fallback в БД, деградация, не падать*

## Углубление

- Внутренности: encoding структур (listpack, skiplist), I/O threads (6.0+), memory fragmentation.
- Client-side caching (RESP3 tracking), near-cache паттерны.
- Отложенные задачи и расписания на sorted set; сравнение с [[task-queues]].
- Valkey/KeyDB/Dragonfly как альтернативы после смены лицензии.

## Ресурсы

- https://redis.io/docs/latest/develop/ (Data types, Patterns: distributed locks, rate limiting)
- Kleppmann, «How to do distributed locking» (про Redlock и fencing tokens)
- https://redis.io/docs/latest/develop/use/patterns/

## Связи

[[resilience-patterns]] · [[sd-building-blocks]] · [[query-optimization-n-plus-1]] · [[auth-security]] · [[task-queues]]

## Мои заметки

