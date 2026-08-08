# 🚀 СУПЕРПОДРОБНЫЙ РОАДМАП НА 3 МЕСЯЦА (Go, High Level + Open Source)

**Старт:** 2 августа 2026 (воскресенье)
**Финиш:** 25 октября 2026
**Режим:** 2-3 часа в день
**Цель:** уровень Senior/Staff в топ-компаниях + умение читать и контрибьютить в open-source

## 📋 Легенда
- **📖 Теория** — чтение/просмотр (1 час)
- **💻 Практика** — написание кода (1-1.5 часа)
- **🔍 Code Reading** — разбор open-source кода (0.5-1 час)
- **🎯 Проект** — работа над своим проектом (выходные 3ч)

---

# 🟢 МЕСЯЦ 1: GO INTERNALS + ADVANCED CONCURRENCY + ЧТЕНИЕ OPEN SOURCE (2 авг — 30 авг)

**Цель месяца:** понять Go изнутри, научиться читать сложный код, освоить продвинутую конкурентность.

---

## НЕДЕЛЯ 1: Go Internals — как работает язык под капотом (2-8 августа)

### Воскресенье (02.08) — Старт
- **📖 Теория (1ч):**
  - Обзор Go runtime: goroutine scheduler (M:N scheduling), G-M-P модель
  - Прочитать: https://go.dev/blog/scheduler
- **💻 Практика (1.5ч):**
  - Написать программу с 100k goroutines, замерить память
  - Использовать `GODEBUG=schedtrace=1000` для трейсинга scheduler'а
  - Построить график: количество goroutines vs CPU usage
- **🔍 Code Reading (0.5ч):**
  - Открыть `src/runtime/proc.go` в Go source — посмотреть на `schedule()` функцию
  - Не пытаться понять всё, просто найти основные блоки

### Понедельник (03.08)
- **📖 Теория (1ч):**
  - Goroutine stacks: начало с 2KB, рост до 1GB
  - Stack growth и copy
  - Прочитать: https://go.dev/blog/stack
- **💻 Практика (1ч):**
  - Написать рекурсивную функцию, замерить рост стека через `runtime.Stack()`
  - Сравнить с heap allocation
- **🔍 Code Reading (1ч):**
  - Разобрать `src/runtime/stack.go` — найти `stackalloc` и `stackfree`
  - Понять, как работает stack pool

### Вторник (04.08)
- **📖 Теория (1ч):**
  - Channels internals: hchan структура, lock-free ring buffer
  - Buffered vs unbuffered — что происходит под капотом
- **💻 Практика (1.5ч):**
  - Написать benchmark: buffered channel vs unbuffered (1M ops)
  - Использовать `runtime/cgo` для понимания overhead
- **🔍 Code Reading (0.5ч):**
  - Открыть `src/runtime/chan.go` — изучить структуру `hchan`
  - Найти `send` и `recv` функции

### Среда (05.08)
- **📖 Теория (1ч):**
  - Garbage Collector: tri-color mark-and-sweep, concurrent marking
  - Write barrier и его стоимость
- **💻 Практика (1.5ч):**
  - Включить `GODEBUG=gctrace=1`
  - Написать benchmark с разными паттернами аллокаций
  - Сравнить GC pause для: много маленьких объектов vs мало больших
- **🔍 Code Reading (0.5ч):**
  - Обзор `src/runtime/mgc.go` — общая структура GC

### Четверг (06.08)
- **📖 Теория (1ч):**
  - Escape analysis: как компилятор решает, куда аллоцировать
  - Команда `go build -gcflags="-m"`
- **💻 Практика (1.5ч):**
  - Написать 10 примеров кода, предсказать escape/no-escape
  - Проверить через `-gcflags="-m -m"` (двойной -m для деталей)
  - Оптимизировать код, чтобы избежать escape в heap
- **🎯 Чек-лист:**
  - [ ] Понимаю, когда значение уходит в heap
  - [ ] Могу оптимизировать код для stack allocation
  - [ ] Знаю про `noescape` и `go:noescape`

### Пятница (07.08)
- **📖 Теория (1ч):**
  - Memory model: happens-before, atomic operations
  - Прочитать: https://go.dev/ref/mem
- **💻 Практика (1.5ч):**
  - Продемонстрировать race condition на уровне memory model
  - Исправить с sync/atomic vs mutex
  - Benchmark: atomic vs mutex (1M ops)
- **🔍 Code Reading (0.5ч):**
  - Разобрать `src/sync/atomic/asm_amd64.s` — как работают атомики на ассемблере

### Суббота (08.08)
- **🎯 Проект (3ч):**
  - Написать **свой простой goroutine scheduler** (упрощенный):
    - Pool of workers
    - Task queue
    - Work stealing (как в Go runtime)
    - Benchmark vs стандартный goroutine pool
  - Измерить: context switch overhead, throughput
- **🔍 Code Reading (1ч):**
  - Прочитать статью "Go scheduler: work stealing" — понять алгоритм

### Воскресенье (09.08)
- **📖 Теория (1ч):**
  - Interface internals: itab, type assertion cost
  - Empty interface vs concrete type
- **💻 Практика (1ч):**
  - Benchmark: interface dispatch vs direct call
  - Измерить стоимость type assertion
  - Оптимизировать hot path, убрав интерфейсы
- **🎯 Чек-лист недели:**
  - [ ] Понимаю G-M-P модель
  - [ ] Знаю, как работает GC и как его тюнить
  - [ ] Умею использовать escape analysis
  - [ ] Понимаю memory model
  - [ ] Могу читать runtime source code

---

## НЕДЕЛЯ 2: Advanced Concurrency Patterns (10-16 августа)

### Понедельник (10.08)
- **📖 Теория (1ч):**
  - Structured Concurrency в Go (предложения, errgroup)
  - Прочитать: https://go.dev/blog/errgroup
- **💻 Практика (1.5ч):**
  - Реализовать свой errgroup с лимитом конкурентности
  - Добавить recovery от паник
  - Написать тесты с race detector
- **🔍 Code Reading (0.5ч):**
  - Разобрать `golang.org/x/sync/errgroup` — исходники

### Вторник (11.08)
- **📖 Теория (1ч):**
  - Semaphore patterns: `golang.org/x/sync/semaphore`
  - Bounded parallelism
- **💻 Практика (1.5ч):**
  - Написать scraper 1000 URL с bounded parallelism (max 50 concurrent)
  - Graceful shutdown с drain
  - Rate limiting на уровне semaphore
- **🎯 Чек-лист:**
  - [ ] Понимаю bounded parallelism
  - [ ] Умею ограничивать concurrency
  - [ ] Знаю, как обрабатывать backpressure

### Среда (12.08)
- **📖 Теория (1ч):**
  - Pipeline patterns: fan-out, fan-in, tee, or-done
  - Прочитать: "Concurrency in Go" Katherine Cox-Buday глава 4
- **💻 Практика (1.5ч):**
  - Реализовать все 4 паттерна
  - Написать pipeline с 5 stages и error propagation
  - Тест: cancellation на любом этапе корректно останавливает всё
- **🔍 Code Reading (0.5ч):**
  - Разобрать pipeline patterns в `hashicorp/vault`

### Четверг (13.08)
- **📖 Теория (1ч):**
  - Context deep dive: values propagation, deadlines
  - Context и memory leaks
- **💻 Практика (1.5ч):**
  - Написать middleware chain с context propagation
  - Продемонстрировать leak: goroutine, который не выходит из-за context
  - Исправить с `context.WithCancelCause` (Go 1.20+)
- **🎯 Чек-лист:**
  - [ ] Понимаю, как context передается через вызовы
  - [ ] Знаю, как избегать context leaks
  - [ ] Умею использовать WithCancelCause

### Пятница (14.08)
- **📖 Теория (1ч):**
  - Worker pools: fixed, dynamic,弹性
  - Прочитать: https://github.com/gammazero/workerpool
- **💻 Практика (1.5ч):**
  - Реализовать свой worker pool:
    - Fixed size
    - Task queue
    - Stats (processed, errors, latency)
    - Graceful shutdown
  - Benchmark vs `gammazero/workerpool`
- **🔍 Code Reading (1ч):**
  - Разобрать `gammazero/workerpool` — как реализован

### Суббота (15.08)
- **🎯 Проект (3ч):**
  - Написать **production-ready worker pool**:
    - Dynamic scaling (min/max workers)
    - Task prioritization
    - Metrics (Prometheus)
    - Graceful shutdown с drain period
    - Recovery от паник в задачах
    - Полное покрытие тестами
  - Опубликовать на GitHub с README

### Воскресенье (16.08)
- **📖 Теория (1ч):**
  - Race detector deep dive: как он работает
  - Data race vs race condition
- **💻 Практика (1.5ч):**
  - Написать 5 сложных race conditions
  - Исправить каждую: mutex, atomic, channel
  - Benchmark overhead race detector'а
- **🎯 Чек-лист недели:**
  - [ ] Умею писать pipeline любой сложности
  - [ ] Понимаю worker pools изнутри
  - [ ] Могу отлаживать race conditions
  - [ ] Знаю structured concurrency

---

## НЕДЕЛЯ 3: ЧТЕНИЕ OPEN SOURCE — УЧИМСЯ ЧИТАТЬ ЧУЖОЙ КОД (17-23 августа)

**Это ключевая неделя.** Senior разработчик отличается от middle тем, что умеет быстро читать и понимать сложный код.

### Понедельник (17.08)
- **📖 Теория (1ч):**
  - Методология чтения кода: "Code Reading" by Diomidis Spinellis
  - Top-down approach: сначала API, потом internals
- **💻 Практика (1.5ч):**
  - Клонировать `gin-gonic/gin`
  - Построить карту: entry points → middleware → handlers
  - Нарисовать диаграмму вызовов для одного HTTP запроса
- **🔍 Code Reading (0.5ч):**
  - Пройти по пути: `gin.Engine.ServeHTTP` → `Context.Next` → handler

### Вторник (18.08)
- **📖 Теория (1ч):**
  - Как использовать `go doc`, `godoc`, IDE для навигации
  - Go to definition, find usages, call hierarchy
- **💻 Практика (1.5ч):**
  - Клонировать `spf13/cobra`
  - Разобрать: как парсятся флаги, как работают subcommands
  - Найти все места, где используется `pflag`
- **🎯 Чек-лист:**
  - [ ] Умею быстро ориентироваться в незнакомом репозитории
  - [ ] Понимаю, как использовать IDE для code navigation
  - [ ] Могу нарисовать call graph

### Среда (19.08)
- **📖 Теория (1ч):**
  - Паттерны проектирования в open-source Go
  - Options pattern, functional options, builder
- **💻 Практика (1.5ч):**
  - Клонировать `go-redis/redis`
  - Найти все варианты Options pattern
  - Реализовать свою библиотеку с functional options
- **🔍 Code Reading (0.5ч):**
  - Разобрать `redis.NewClient` — как собирается конфигурация

### Четверг (20.08)
- **📖 Теория (1ч):**
  - Error handling patterns в open-source
  - Sentinel errors, error types, wrapping
- **💻 Практика (1.5ч):**
  - Клонировать `jackc/pgx`
  - Изучить, как обрабатываются ошибки БД
  - Найти все места, где используется `errors.Is` и `errors.As`
- **🎯 Чек-лист:**
  - [ ] Понимаю functional options pattern
  - [ ] Умею проектировать error handling
  - [ ] Знаю best practices из open-source

### Пятница (21.08)
- **📖 Теория (1ч):**
  - Testing patterns в open-source
  - Table-driven tests, test helpers, test fixtures
- **💻 Практика (1.5ч):**
  - Клонировать `stretchr/testify`
  - Разобрать: как реализованы assertions
  - Написать свой assertion с красивым diff
- **🔍 Code Reading (0.5ч):**
  - Изучить test helpers в `kubernetes/kubernetes`

### Суббота (22.08)
- **🎯 Проект (3ч):**
  - **Code Reading Challenge**: разобрать `prometheus/prometheus`:
    - Найти entry point (main.go)
    - Понять, как работает scrape loop
    - Разобрать TSDB (time-series database) на верхнем уровне
    - Написать свой мини-анализ в markdown (500 слов)
  - Опубликовать разбор как статью

### Воскресенье (23.08)
- **📖 Теория (1ч):**
  - Как читать системный код: `etcd`, `consul`
  - Понимание consensus алгоритмов через код
- **💻 Практика (1.5ч):**
  - Клонировать `etcd-io/etcd`
  - Найти реализацию Raft (через `go.etcd.io/raft`)
  - Понять: leader election, log replication на уровне кода
- **🎯 Чек-лист недели:**
  - [ ] Умею читать код уровня production framework
  - [ ] Понимаю паттерны, которые используют в open-source
  - [ ] Могу разобрать сложную систему на части
  - [ ] Написал свой code review/разбор

---

## НЕДЕЛЯ 4: Performance + Profiling + Zero-Allocation Code (24-30 августа)

### Понедельник (24.08)
- **📖 Теория (1ч):**
  - pprof: CPU, memory, goroutine, mutex, block profiles
  - Прочитать: https://go.dev/doc/diagnostics
- **💻 Практика (1.5ч):**
  - Написать HTTP сервер с искусственной нагрузкой
  - Снять все 5 видов профилей
  - Найти bottleneck и исправить
- **🔍 Code Reading (0.5ч):**
  - Разобрать, как pprof используется в `grafana/loki`

### Вторник (25.08)
- **📖 Теория (1ч):**
  - Benchmarks: `testing.B`, `b.ReportAllocs()`, `b.RunParallel`
  - Sub-benchmarks и сравнение
- **💻 Практика (1.5ч):**
  - Написать benchmark suite для 3 реализаций rate limiter'а
  - Сравнить: mutex, atomic, channel-based
  - Найти самую быструю реализацию
- **🎯 Чек-лист:**
  - [ ] Умею писать benchmarks
  - [ ] Понимаю `ReportAllocs` и `RunParallel`
  - [ ] Могу сравнивать реализации

### Среда (26.08)
- **📖 Теория (1ч):**
  - Zero-allocation techniques: `sync.Pool`, `[]byte` reuse
  - `strings.Builder` vs `bytes.Buffer`
- **💻 Практика (1.5ч):**
  - Написать JSON парсер с zero allocations
  - Использовать `sync.Pool` для буферов
  - Benchmark: до/после оптимизации
- **🔍 Code Reading (0.5ч):**
  - Разобрать `encoding/json` — как минимизируются аллокации

### Четверг (27.08)
- **📖 Теория (1ч):**
  - `go tool trace`: visualization of goroutines, GC, syscalls
  - Когда использовать trace vs pprof
- **💻 Практика (1.5ч):**
  - Снять trace для concurrent приложения
  - Найти: goroutine leaks, scheduler latency, syscall overhead
  - Оптимизировать на основе trace
- **🎯 Чек-лист:**
  - [ ] Умею использовать `go tool trace`
  - [ ] Понимаю разницу trace vs pprof
  - [ ] Могу найти scheduler issues

### Пятница (28.08)
- **📖 Теория (1ч):**
  - Compiler optimizations: inlining, devirtualization
  - `go:noinline`, `go:linkname`, `go:nosplit`
- **💻 Практика (1.5ч):**
  - Проанализировать inlining decisions через `-gcflags="-m"`
  - Написать код, который компилятор не может заинлайнить
  - Исправить через restructuring
- **🔍 Code Reading (0.5ч):**
  - Разобрать `go:linkname` usage в `runtime` package

### Суббота (29.08)
- **🎯 Проект (3ч):**
  - **Performance Challenge**: оптимизировать HTTP middleware до 0 аллокаций:
    - JWT validation
    - Rate limiting
    - Request logging
    - Benchmark: 1M requests
    - Цель: 0 allocs/op, p99 < 50μs
  - Опубликовать на GitHub с бенчмарками

### Воскресенье (30.08)
- **📖 Теория (1ч):**
  - Fuzzing в Go: `testing.F`
  - Property-based testing
- **💻 Практика (1.5ч):**
  - Написать fuzz test для парсера
  - Найти баг через fuzzing
  - Написать property-based тест с `rapid` библиотекой
- **🎯 Чек-лист недели и месяца:**
  - [ ] Умею профилировать Go код (pprof, trace)
  - [ ] Пишу benchmarks и оптимизирую
  - [ ] Знаю zero-allocation техники
  - [ ] Понимаю compiler optimizations
  - [ ] Умею читать любой open-source код
  - [ ] Понимаю Go internals

---

# 🔵 МЕСЯЦ 2: DISTRIBUTED SYSTEMS + PRODUCTION PATTERNS (31 авг — 27 сен)

**Цель месяца:** освоить паттерны, которые используются в топ-компаниях, научиться писать отказоустойчивые системы.

---

## НЕДЕЛЯ 5: Production-Grade HTTP/gRPC сервисы (31 авг — 6 сен)

### Понедельник (31.08)
- **📖 Теория (1ч):**
  - HTTP/2 и HTTP/3 в Go: `net/http` internals
  - Connection pooling, keep-alive
- **💻 Практика (1.5ч):**
  - Настроить HTTP client с optimal settings
  - Benchmark: default vs tuned client
  - Измерить: connections per host, idle timeout
- **🔍 Code Reading (0.5ч):**
  - Разобрать `net/http/transport.go` — как работает connection pool

### Вторник (01.09)
- **📖 Теория (1ч):**
  - gRPC production patterns: keepalive, max message size, retries
  - Прочитать: https://github.com/grpc/grpc-go/blob/master/Documentation
- **💻 Практика (1.5ч):**
  - Настроить gRPC client с retry policy
  - Реализовать circuit breaker interceptor
  - Тест: simulate network failures
- **🎯 Чек-лист:**
  - [ ] Понимаю HTTP/2 internals
  - [ ] Умею настраивать gRPC для production
  - [ ] Знаю keepalive и retry policies

### Среда (02.09)
- **📖 Теория (1ч):**
  - Graceful shutdown в Go: `signal.NotifyContext`
  - Drain period, in-flight requests
- **💻 Практика (1.5ч):**
  - Реализовать graceful shutdown для HTTP + gRPC + Kafka
  - Тест: SIGTERM во время обработки запроса
  - Убедиться, что все запросы завершены
- **🔍 Code Reading (0.5ч):**
  - Разобрать shutdown в `kubernetes/kubernetes`

### Четверг (03.09)
- **📖 Теория (1ч):**
  - Middleware patterns: chain, decorator
  - Прочитать: https://github.com/justinas/alice
- **💻 Практика (1.5ч):**
  - Написать middleware chain: auth → logging → metrics → recovery → handler
  - Реализовать middleware для injection dependencies
  - Тест: порядок выполнения, error propagation
- **🎯 Чек-лист:**
  - [ ] Понимаю middleware patterns
  - [ ] Умею писать production middleware
  - [ ] Знаю graceful shutdown best practices

### Пятница (04.09)
- **📖 Теория (1ч):**
  - Configuration management: env, flags, config files, remote config
  - Прочитать: `spf13/viper` deep dive
- **💻 Практика (1.5ч):**
  - Реализовать hot reload конфигурации
  - Использовать feature flags (Unleash/openfeature)
  - Тест: изменение конфига без restart
- **🔍 Code Reading (0.5ч):**
  - Разобрать config в `etcd-io/etcd`

### Суббота (05.09)
- **🎯 Проект (3ч):**
  - Написать **production-ready HTTP сервис**:
    - Graceful shutdown
    - Middleware chain
    - Hot reload config
    - Feature flags
    - Health checks (liveness, readiness)
    - OpenAPI spec
  - Тест: 10k RPS с graceful shutdown

### Воскресенье (06.09)
- **📖 Теория (1ч):**
  - API design: REST vs gRPC vs GraphQL
  - Idempotency в API
- **💻 Практика (1.5ч):**
  - Реализовать idempotent API с `Idempotency-Key` header
  - Хранить ключи в Redis с TTL
  - Тест: повторный запрос возвращает тот же результат
- **🎯 Чек-лист недели:**
  - [ ] Понимаю HTTP/2, gRPC production patterns
  - [ ] Умею реализовывать graceful shutdown
  - [ ] Знаю middleware и config patterns
  - [ ] Могу писать idempotent API

---

## НЕДЕЛЯ 6: Database Deep Dive + Patterns (7-13 сентября)

### Понедельник (07.09)
- **📖 Теория (1ч):**
  - `pgx` vs `database/sql`: когда что использовать
  - Connection pool tuning
- **💻 Практика (1.5ч):**
  - Настроить pgxpool с оптимальными параметрами
  - Benchmark: pool size vs throughput
  - Найти optimal pool size для своей нагрузки
- **🔍 Code Reading (0.5ч):**
  - Разобрать `jackc/pgx/pgxpool` — как работает pool

### Вторник (08.09)
- **📖 Теория (1ч):**
  - Transaction patterns: savepoints, advisory locks
  - SKIP LOCKED для очередей на PostgreSQL
- **💻 Практика (1.5ч):**
  - Реализовать очередь задач на PostgreSQL с SKIP LOCKED
  - Benchmark vs Redis queue
  - Тест: 100 workers, 10k tasks
- **🎯 Чек-лист:**
  - [ ] Понимаю pgxpool tuning
  - [ ] Умею использовать SKIP LOCKED
  - [ ] Знаю advisory locks

### Среда (09.09)
- **📖 Теория (1ч):**
  - Миграции: expand-contract pattern
  - Zero-downtime schema changes
- **💻 Практика (1.5ч):**
  - Написать миграцию: переименование колонки без downtime
  - Реализовать dual-write, backfill, switchover
  - Rollback скрипт
- **🔍 Code Reading (0.5ч):**
  - Разобрать миграции в `golang-migrate/migrate`

### Четверг (10.09)
- **📖 Теория (1ч):**
  - Read replicas и CQRS в Go
  - Routing queries к нужной реплике
- **💻 Практика (1.5ч):**
  - Реализовать router: writes → master, reads → replica
  - Handle replication lag
  - Тест: 80% reads, 20% writes
- **🎯 Чек-лист:**
  - [ ] Понимаю expand-contract migrations
  - [ ] Умею реализовывать CQRS
  - [ ] Знаю, как обрабатывать replication lag

### Пятница (11.09)
- **📖 Теория (1ч):**
  - SQL query optimization в Go
  - `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` парсинг
- **💻 Практика (1.5ч):**
  - Написать утилиту для парсинга EXPLAIN в Go
  - Автоматически находить: sequential scans, missing indexes
  - Интегрировать в CI
- **🔍 Code Reading (0.5ч):**
  - Разобрать query optimization в `cockroachdb/cockroach`

### Суббота (12.09)
- **🎯 Проект (3ч):**
  - Написать **production-ready repository layer**:
    - pgxpool с оптимальными настройками
    - Read/write splitting
    - Transaction management
    - Query analyzer (EXPLAIN parser)
    - Миграции с zero-downtime
  - Тест: 10k RPS, p99 < 10ms

### Воскресенье (13.09)
- **📖 Теория (1ч):**
  - NoSQL в Go: когда использовать
  - MongoDB, DynamoDB, ScyllaDB drivers
- **💻 Практика (1.5ч):**
  - Написать benchmark: PostgreSQL vs MongoDB для разных workload'ов
  - Определить, когда что использовать
  - Реализовать repository abstraction
- **🎯 Чек-лист недели:**
  - [ ] Понимаю pgxpool tuning
  - [ ] Умею делать zero-downtime migrations
  - [ ] Знаю CQRS и read/write splitting
  - [ ] Могу оптимизировать SQL queries

---

## НЕДЕЛЯ 7: Kafka + Event-Driven Architecture (14-20 сентября)

### Понедельник (14.09)
- **📖 Теория (1ч):**
  - `segmentio/kafka-go` vs `confluent-kafka-go` vs `IBM/sarama`
  - Когда что использовать
- **💻 Практика (1.5ч):**
  - Написать benchmark 3 библиотек
  - Найти оптимальную для своего случая
  - Измерить: throughput, latency, memory
- **🔍 Code Reading (0.5ч):**
  - Разобрать `segmentio/kafka-go` internals

### Вторник (15.09)
- **📖 Теория (1ч):**
  - Outbox pattern: implementation strategies
  - Polling publisher vs CDC (Debezium)
- **💻 Практика (1.5ч):**
  - Реализовать outbox с polling publisher
  - Настроить transactional publishing
  - Тест: БД commit → Kafka publish атомарно
- **🎯 Чек-лист:**
  - [ ] Понимаю outbox pattern
  - [ ] Умею реализовывать polling publisher
  - [ ] Знаю CDC подход

### Среда (16.09)
- **📖 Теория (1ч):**
  - Consumer patterns: idempotency, exactly-once
  - Consumer group rebalancing
- **💻 Практика (1.5ч):**
  - Реализовать idempotent consumer с Redis
  - Обработать rebalancing корректно
  - Тест: consumer crash → restart → no duplicates
- **🔍 Code Reading (0.5ч):**
  - Разобрать consumer patterns в `shopify/sarama`

### Четверг (17.09)
- **📖 Теория (1ч):**
  - Dead Letter Queue patterns
  - Retry strategies: exponential, jitter
- **💻 Практика (1.5ч):**
  - Реализовать DLQ с retry
  - Написать admin tool для replay
  - Тест: 1% failed messages → DLQ → replay
- **🎯 Чек-лист:**
  - [ ] Понимаю idempotent consumers
  - [ ] Умею реализовывать DLQ
  - [ ] Знаю retry strategies

### Пятница (18.09)
- **📖 Теория (1ч):**
  - Schema evolution: Avro, Protobuf, JSON Schema
  - Schema registry
- **💻 Практика (1.5ч):**
  - Настроить Confluent Schema Registry
  - Реализовать backward/forward compatibility
  - Тест: producer v2 → consumer v1 работает
- **🔍 Code Reading (0.5ч):**
  - Разобрать schema evolution в `linkedin/schema-evolution-samples`

### Суббота (19.09)
- **🎯 Проект (3ч):**
  - Написать **production event pipeline**:
    - Outbox pattern
    - Idempotent consumers
    - DLQ с retry
    - Schema registry
    - Metrics для pipeline
  - Тест: 100k events/sec, 0 duplicates

### Воскресенье (20.09)
- **📖 Теория (1ч):**
  - Event sourcing basics
  - CQRS с Kafka
- **💻 Практика (1.5ч):**
  - Реализовать event-sourced aggregate
  - Snapshot для оптимизации
  - Тест: rebuild state from events
- **🎯 Чек-лист недели:**
  - [ ] Понимаю Kafka production patterns
  - [ ] Умею реализовывать outbox, DLQ, idempotency
  - [ ] Знаю schema evolution
  - [ ] Могу построить event pipeline

---

## НЕДЕЛЯ 8: Distributed Patterns + Resilience (21-27 сентября)

### Понедельник (21.09)
- **📖 Теория (1ч):**
  - Circuit breaker: `sony/gobreaker` deep dive
  - States: closed, open, half-open
- **💻 Практика (1.5ч):**
  - Реализовать свой circuit breaker
  - Интегрировать с HTTP и gRPC clients
  - Тест: simulate downstream failures
- **🔍 Code Reading (0.5ч):**
  - Разобрать `sony/gobreaker` source

### Вторник (22.09)
- **📖 Теория (1ч):**
  - Bulkhead pattern: изоляция failures
  - Timeout, retry, fallback
- **💻 Практика (1.5ч):**
  - Реализовать bulkhead для разных downstream сервисов
  - Настроить fallback responses
  - Тест: один сервис падает, другие работают
- **🎯 Чек-лист:**
  - [ ] Понимаю circuit breaker
  - [ ] Умею реализовывать bulkhead
  - [ ] Знаю fallback patterns

### Среда (23.09)
- **📖 Теория (1ч):**
  - Saga pattern: choreography vs orchestration
  - Прочитать: "Microservices Patterns" Chris Richardson глава 4
- **💻 Практика (1.5ч):**
  - Реализовать saga orchestrator
  - State machine для управления шагами
  - Compensation logic
- **🔍 Code Reading (0.5ч):**
  - Разобрать saga в `temporalio/temporal` samples

### Четверг (24.09)
- **📖 Теория (1ч):**
  - Distributed locking: Redlock, etcd leases
  - Когда использовать, когда избегать
- **💻 Практика (1.5ч):**
  - Реализовать distributed lock на Redis
  - Реализовать на etcd leases
  - Benchmark и сравнение
- **🎯 Чек-лист:**
  - [ ] Понимаю saga pattern
  - [ ] Умею реализовывать orchestrator
  - [ ] Знаю distributed locking

### Пятница (25.09)
- **📖 Теория (1ч):**
  - Service discovery: Consul, etcd, Kubernetes
  - Client-side vs server-side LB
- **💻 Практика (1.5ч):**
  - Реализовать client-side discovery с Consul
  - Health checking и deregistration
  - Тест: service instance падает → LB убирает его
- **🔍 Code Reading (0.5ч):**
  - Разобрать discovery в `hashicorp/consul`

### Суббота (26.09)
- **🎯 Проект (3ч):**
  - Написать **resilient microservice framework**:
    - Circuit breaker
    - Bulkhead
    - Retry с backoff
    - Timeout
    - Fallback
    - Service discovery
  - Тест: 10 сервисов, simulate failures

### Воскресенье (27.09)
- **📖 Теория (1ч):**
  - Consistency patterns: eventual, strong, causal
  - Vector clocks, CRDTs
- **💻 Практика (1.5ч):**
  - Реализовать simple CRDT (G-Counter)
  - Тест: concurrent updates → convergence
- **🎯 Чек-лист недели и месяца:**
  - [ ] Понимаю circuit breaker, bulkhead
  - [ ] Умею реализовывать saga
  - [ ] Знаю distributed locking
  - [ ] Понимаю service discovery
  - [ ] Могу построить resilient систему

---

# 🔴 МЕСЯЦ 3: OBSERVABILITY + HIGH LOAD + OPEN SOURCE CONTRIBUTION (28 сен — 25 окт)

**Цель месяца:** observability, high load, и главное — начать контрибьютить в open-source.

---

## НЕДЕЛЯ 9: Observability Deep Dive (28 сен — 4 окт)

### Понедельник (28.09)
- **📖 Теория (1ч):**
  - OpenTelemetry Go: traces, metrics, logs
  - Прочитать: https://opentelemetry.io/docs/languages/go/
- **💻 Практика (1.5ч):**
  - Интегрировать OTel в свой сервис
  - Настроить exporter в Jaeger, Prometheus, Loki
  - Тест: увидеть полный трейс
- **🔍 Code Reading (0.5ч):**
  - Разобрать OTel integration в `grafana/loki`

### Вторник (29.09)
- **📖 Теория (1ч):**
  - Context propagation через HTTP, gRPC, Kafka
  - W3C Trace Context, B3
- **💻 Практика (1.5ч):**
  - Реализовать propagation через все протоколы
  - Тест: трейс проходит через 5 сервисов
  - Проверить в Jaeger
- **🎯 Чек-лист:**
  - [ ] Понимаю OpenTelemetry
  - [ ] Умею пробрасывать context
  - [ ] Знаю W3C Trace Context

### Среда (30.09)
- **📖 Теория (1ч):**
  - Prometheus metrics: counter, gauge, histogram, summary
  - RED и USE методы
- **💻 Практика (1.5ч):**
  - Добавить метрики для каждого эндпоинта
  - Настроить histogram buckets правильно
  - Создать Grafana dashboard
- **🔍 Code Reading (0.5ч):**
  - Разобрать metrics в `prometheus/prometheus`

### Четверг (01.10)
- **📖 Теория (1ч):**
  - Structured logging: `uber-go/zap`, `rs/zerolog`
  - Log levels, sampling, correlation
- **💻 Практика (1.5ч):**
  - Настроить zap с correlation_id
  - Реализовать log sampling для high-load
  - Интегрировать с Loki
- **🎯 Чек-лист:**
  - [ ] Понимаю Prometheus metrics
  - [ ] Умею настраивать structured logging
  - [ ] Знаю RED и USE методы

### Пятница (02.10)
- **📖 Теория (1ч):**
  - SLI/SLO/SLA: определение и реализация
  - Error budgets
- **💻 Практика (1.5ч):**
  - Определить SLI для своего сервиса
  - Настроить SLO alerting
  - Реализовать error budget tracking
- **🔍 Code Reading (0.5ч):**
  - Разобрать SLO в `google/slo-generator`

### Суббота (03.10)
- **🎯 Проект (3ч):**
  - Добавить **полную observability** в свой сервис:
    - OpenTelemetry traces
    - Prometheus metrics
    - Structured logs с correlation
    - SLO dashboards
    - Alerting rules
  - Тест: полный observability stack

### Воскресенье (04.10)
- **📖 Теория (1ч):**
  - Profiling в production: continuous profiling
  - Pyroscope, Parca
- **💻 Практика (1.5ч):**
  - Настроить continuous profiling
  - Найти bottleneck в production-like нагрузке
  - Оптимизировать
- **🎯 Чек-лист недели:**
  - [ ] Понимаю observability stack
  - [ ] Умею настраивать OpenTelemetry
  - [ ] Знаю SLI/SLO
  - [ ] Могу профилировать в production

---

## НЕДЕЛЯ 10: High Load Optimization (5-11 октября)

### Понедельник (05.10)
- **📖 Теория (1ч):**
  - Load testing: `grafana/k6`, `tsenart/vegeta`
  - Load testing patterns
- **💻 Практика (1.5ч):**
  - Написать k6 скрипт для своего сервиса
  - Провести: smoke, load, stress, soak tests
  - Найти breaking point
- **🔍 Code Reading (0.5ч):**
  - Разобрать k6 internals

### Вторник (06.10)
- **📖 Теория (1ч):**
  - Kernel tuning для high load
  - `sysctl`, file descriptors, TCP tuning
- **💻 Практика (1.5ч):**
  - Настроить Linux kernel для high load
  - Измерить: до/после tuning
  - Benchmark: 50k concurrent connections
- **🎯 Чек-лист:**
  - [ ] Понимаю load testing patterns
  - [ ] Умею тюнить Linux kernel
  - [ ] Знаю TCP tuning

### Среда (07.10)
- **📖 Теория (1ч):**
  - Connection pooling optimization
  - Keep-alive, idle connections
- **💻 Практика (1.5ч):**
  - Оптимизировать HTTP client pool
  - Настроить keep-alive правильно
  - Benchmark: 10k RPS
- **🔍 Code Reading (0.5ч):**
  - Разобрать connection pool в `valyala/fasthttp`

### Четверг (08.10)
- **📖 Теория (1ч):**
  - Caching strategies: multi-level cache
  - L1 (in-process), L2 (Redis), L3 (CDN)
- **💻 Практика (1.5ч):**
  - Реализовать multi-level cache
  - Invalidation между уровнями
  - Benchmark: cache hit rate
- **🎯 Чек-лист:**
  - [ ] Понимаю connection pooling
  - [ ] Умею реализовывать multi-level cache
  - [ ] Знаю cache invalidation

### Пятница (09.10)
- **📖 Теория (1ч):**
  - Async processing: worker pools, background jobs
  - `hibiken/asynq` — Redis-based queue
- **💻 Практика (1.5ч):**
  - Реализовать background job system
  - Priority queues, retry, DLQ
  - Тест: 100k jobs/hour
- **🔍 Code Reading (0.5ч):**
  - Разобрать `hibiken/asynq` source

### Суббота (10.10)
- **🎯 Проект (3ч):**
  - **High Load Challenge**: оптимизировать сервис до 50k RPS:
    - Kernel tuning
    - Connection pooling
    - Multi-level cache
    - Background processing
    - Load testing с k6
  - Цель: p99 < 50ms при 50k RPS

### Воскресенье (11.10)
- **📖 Теория (1ч):**
  - Chaos engineering: `litmuschaos`, `chaos-mesh`
  - Fault injection
- **💻 Практика (1.5ч):**
  - Настроить chaos testing
  - Inject: network latency, pod failures
  - Тест: система восстанавливается
- **🎯 Чек-лист недели:**
  - [ ] Понимаю load testing
  - [ ] Умею тюнить kernel
  - [ ] Знаю chaos engineering
  - [ ] Могу оптимизировать до 50k RPS

---

## НЕДЕЛЯ 11: OPEN SOURCE CONTRIBUTION — НАЧИНАЕМ КОНТРИБЬЮТИТЬ (12-18 октября)

**Это ключевая неделя для перехода на новый уровень.**

### Понедельник (12.10)
- **📖 Теория (1ч):**
  - Как найти свой первый issue: "good first issue", documentation
  - Прочитать: https://opensource.guide/how-to-contribute/
- **💻 Практика (1.5ч):**
  - Выбрать 3 проекта, которыми пользуешься
  - Найти 5 issues с лейблом "good first issue"
  - Проанализировать: какой подходит тебе
- **🔍 Code Reading (0.5ч):**
  - Изучить CONTRIBUTING.md в выбранных проектах

### Вторник (13.10)
- **📖 Теория (1ч):**
  - Git workflow для open-source: fork, branch, PR
  - Commit messages, DCO sign-off
- **💻 Практика (1.5ч):**
  - Настроить fork для выбранного проекта
  - Создать branch, сделать первый маленький PR
  - Исправить: typo, documentation, small bug
- **🎯 Чек-лист:**
  - [ ] Понимаю open-source workflow
  - [ ] Умею делать fork и PR
  - [ ] Знаю commit conventions

### Среда (14.10)
- **📖 Теория (1ч):**
  - Как писать хорошие PR: description, tests, screenshots
  - Communication with maintainers
- **💻 Практика (1.5ч):**
  - Написать идеальный PR description
  - Добавить тесты для своих изменений
  - Ответить на review comments
- **🔍 Code Reading (0.5ч):**
  - Изучить merged PRs в проекте — что делает PR хорошим

### Четверг (15.10)
- **📖 Теория (1ч):**
  - Finding bigger issues: features, refactoring
  - Proposing changes через RFC/issue
- **💻 Практика (1.5ч):**
  - Найти issue среднего размера
  - Написать proposal в issue (если нужно)
  - Начать реализацию
- **🎯 Чек-лист:**
  - [ ] Умею писать хорошие PR
  - [ ] Понимаю communication с maintainers
  - [ ] Могу предложить feature

### Пятница (16.10)
- **📖 Теория (1ч):**
  - Testing в open-source: unit, integration, e2e
  - CI/CD в open-source projects
- **💻 Практика (1.5ч):**
  - Написать тесты для своих изменений
  - Прогнать через CI
  - Исправить все failures
- **🔍 Code Reading (0.5ч):**
  - Разобрать CI в проекте (GitHub Actions, CircleCI)

### Суббота (17.10)
- **🎯 Проект (3ч):**
  - **Open Source Challenge**: сделать 3 PR'а:
    - 1 documentation fix
    - 1 small bug fix
    - 1 feature или refactoring
  - Цель: все 3 PR'а merged
  - Получить feedback от maintainers

### Воскресенье (18.10)
- **📖 Теория (1ч):**
  - Maintainer experience: как стать maintainer
  - Building community
- **💻 Практика (1.5ч):**
  - Написать статью о своем опыте контрибьюции
  - Опубликовать на Хабр/Medium/Dev.to
  - Ответить на вопросы в issues других людей
- **🎯 Чек-лист недели:**
  - [ ] Сделал 3+ PR'а в open-source
  - [ ] Понимаю maintainer workflow
  - [ ] Написал статью о своем опыте
  - [ ] Начал строить репутацию

---

## НЕДЕЛЯ 12: ФИНАЛЬНЫЙ ПРОЕКТ + ПОДГОТОВКА К СОБЕСЕДОВАНИЯМ (19-25 октября)

### Понедельник (19.10)
- **📖 Теория (1ч):**
  - System design interviews: framework
  - Прочитать: "System Design Interview" Alex Xu глава 1
- **💻 Практика (1.5ч):**
  - Решить 3 system design задачи:
    - Design URL shortener
    - Design rate limiter
    - Design notification system
  - Нарисовать диаграммы

### Вторник (20.10)
- **📖 Теория (1ч):**
  - Go-specific interview questions
  - Internals, concurrency, memory model
- **💻 Практика (1.5ч):**
  - Решить 20 Go-specific вопросов
  - Написать код для каждого
  - Объяснить устно (записать на видео)
- **🔍 Code Reading (0.5ч):**
  - Разобрать сложные вопросы из Go interviews

### Среда (21.10)
- **📖 Теория (1ч):**
  - Distributed systems interview questions
  - Consensus, consistency, availability
- **💻 Практика (1.5ч):**
  - Решить 10 distributed systems задач
  - Реализовать: consensus, distributed lock, leader election
  - Объяснить trade-offs

### Четверг (22.10)
- **📖 Теория (1ч):**
  - Behavioral interviews: STAR method
  - Leadership principles (Amazon)
- **💻 Практика (1.5ч):**
  - Подготовить 10 stories по STAR
  - Записать ответы на видео
  - Посмотреть и улучшить

### Пятница (23.10)
- **🎯 ФИНАЛЬНЫЙ ПРОЕКТ (3ч):**
  - **Production-grade e-commerce платформа** (финальная сборка всего):
    - Микросервисы: Auth, Product, Order, Payment, Inventory, Notification
    - Go 1.23+, gRPC, Kafka, PostgreSQL, Redis
    - Clean Architecture + DDD
    - Outbox pattern + Saga
    - OpenTelemetry + Prometheus + Jaeger
    - Circuit breaker + retry + DLQ
    - Graceful shutdown + health checks
    - Zero-allocation hot paths
    - 50k RPS, p99 < 50ms
    - 90%+ test coverage
    - OpenAPI + gRPC documentation
    - Kubernetes manifests
    - **Опубликовать на GitHub с подробным README**

### Суббота (24.10)
- **💻 Практика (3ч):**
  - Финальная полировка проекта
  - Написать подробный README:
    - Architecture diagram
    - How to run
    - Performance benchmarks
    - Open source contributions made
  - Записать demo video (5 минут)

### Воскресенье (25.10) — ФИНИШ
- **📖 Теория (1ч):**
  - Построить roadmap дальнейшего развития
  - Staff engineer path
- **💻 Практика (1.5ч):**
  - Написать итоговую статью: "3 месяца Go intensive — что я изучил"
  - Обновить резюме и LinkedIn
  - Начать откликаться на вакансии уровня Senior/Staff
- **🎯 ФИНАЛЬНЫЙ ЧЕК-ЛИСТ:**
  - [ ] Понимаю Go internals (scheduler, GC, memory model)
  - [ ] Умею профилировать и оптимизировать Go код
  - [ ] Пишу zero-allocation код
  - [ ] Знаю все production patterns (outbox, saga, circuit breaker)
  - [ ] Умею читать сложный open-source код
  - [ ] Сделал 3+ PR'а в open-source
  - [ ] Построил production-grade микросервисную систему
  - [ ] Готов к собеседованиям в топ-компании

---

# 📚 ОБЯЗАТЕЛЬНЫЕ РЕСУРСЫ НА ВСЕ 3 МЕСЯЦА

## Книги (читать параллельно, по 20 страниц в день):
1. **"100 Go Mistakes and How to Avoid Them"** — Teiva Harsanyi
2. **"Concurrency in Go"** — Katherine Cox-Buday
3. **"Designing Data-Intensive Applications"** — Martin Kleppmann
4. **"Microservices Patterns"** — Chris Richardson

## Open-source проекты для чтения (в порядке сложности):
1. **Уровень 1:** `gin-gonic/gin`, `spf13/cobra`, `stretchr/testify`
2. **Уровень 2:** `go-redis/redis`, `jackc/pgx`, `segmentio/kafka-go`
3. **Уровень 3:** `prometheus/prometheus`, `etcd-io/etcd`, `hashicorp/consul`
4. **Уровень 4:** `kubernetes/kubernetes`, `cockroachdb/cockroach`, `moby/moby`

## Open-source проекты для контрибьюции (рекомендации):
- **Для начала:** `uber-go/zap`, `spf13/viper`, `go-chi/chi`
- **Средний уровень:** `segmentio/kafka-go`, `go-redis/redis`, `hibiken/asynq`
- **Продвинутый:** `prometheus/prometheus`, `etcd-io/etcd`, `open-telemetry/opentelemetry-go`

## YouTube:
- **GopherCon** — доклады (смотреть 2 в неделю)
- **Gopher Academy** — туториалы
- **Anthony GG** — Go tutorials
- **Bitfield Consulting** — Go code review

## Блоги:
- **Go Blog** (go.dev/blog)
- **Uber Engineering**
- **Cloudflare Blog**
- **Discord Engineering**

## Подкасты:
- **Go Time**
- **Software Engineering Daily**

---

# 🎯 МЕТРИКИ УСПЕХА ПОСЛЕ 3 МЕСЯЦЕВ

**Технические навыки:**
- [ ] Могу объяснить, как работает Go scheduler
- [ ] Понимаю GC и умею его тюнить
- [ ] Пишу zero-allocation код
- [ ] Умею профилировать (pprof, trace)
- [ ] Знаю все concurrency patterns
- [ ] Могу построить микросервисную систему с нуля
- [ ] Умею делать zero-downtime migrations
- [ ] Знаю outbox, saga, circuit breaker
- [ ] Понимаю observability stack
- [ ] Могу оптимизировать до 50k+ RPS

**Open-source:**
- [ ] Сделал 3+ PR'а в известные проекты
- [ ] Умею читать код уровня Kubernetes, etcd, Prometheus
- [ ] Написал 2+ статьи о разборе open-source кода
- [ ] Понимаю maintainer workflow

**Soft skills:**
- [ ] Готов к system design собеседованиям
- [ ] Умею объяснять сложные вещи просто
- [ ] Имею портфолио production-grade проектов
- [ ] Готов к уровню Senior/Staff в топ-компаниях

---

# 💡 КЛЮЧЕВЫЕ ПРИНЦИПЫ

1. **Каждый день код** — даже 30 минут лучше, чем пропуск
2. **Читай open-source каждый день** — минимум 30 минут чтения чужого кода
3. **Пиши статьи** — объяснение = понимание
4. **Делай PR'а** — репутация в open-source открывает двери
5. **Benchmark всё** — цифры важнее ощущений
6. **Профилируй** — без pprof оптимизация вслепую
7. **Тестируй** — без тестов код не существует
8. **Документируй** — README важнее кода для open-source

---

# 🚀 ЧТО ДЕЛАТЬ ПРЯМО СЕЙЧАС (2 августа 2026)

1. **Сегодня (воскресенье, 2 августа):**
   - Прочитать про Go scheduler (1ч)
   - Написать программу с 100k goroutines (1.5ч)
   - Открыть `src/runtime/proc.go` и найти `schedule()` (0.5ч)

2. **Завтра (понедельник, 3 августа):**
   - Изучить goroutine stacks (1ч)
   - Написать рекурсивную функцию с замером стека (1ч)
   - Разобрать `src/runtime/stack.go` (1ч)

3. **На этой неделе:**
   - Пройти всю Неделю 1
   - К концу недели ты будешь понимать Go internals лучше, чем 90% Go разработчиков

Удачи! Через 3 месяца ты будешь другим разработчиком. 🎯