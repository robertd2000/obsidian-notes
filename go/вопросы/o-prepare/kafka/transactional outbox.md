На собеседовании transactional outbox лучше объяснять не как “паттерн для Kafka”, а как решение проблемы **dual write**.

Проблема выглядит так:

есть сервис, который:

1. сохраняет данные в БД
    
2. публикует событие в Apache Kafka или RabbitMQ
    

И эти две операции находятся в разных системах.

---

Например:

```text
BEGIN;
UPDATE orders SET status='paid';
COMMIT;

publish(order_paid)
```

Что если:

- БД commit прошёл
    
- а publish в Kafka упал?
    

Тогда:

- данные изменились
    
- события нет
    
- системы разъехались
    

Обратная ситуация тоже возможна:

- событие отправили
    
- транзакция БД откатилась
    

Тогда downstream сервисы получили событие о том, чего на самом деле нет.

Это и есть проблема dual write.

---

Transactional outbox решает её так:

вместо прямой отправки в Kafka сервис пишет событие в специальную outbox table внутри той же транзакции БД.

Например:

```sql
BEGIN;

UPDATE orders
SET status = 'paid';

INSERT INTO outbox_events (
  event_id,
  event_type,
  payload
)
VALUES (...);

COMMIT;
```

Теперь:

- либо commit’ится и бизнес-данные, и событие
    
- либо не commit’ится ничего
    

То есть появляется атомарность.

---

Дальше отдельный process:

- polling worker
    
- CDC connector
    
- relay service
    

читает outbox table и публикует события в Kafka.

После успешной публикации:

- помечает событие как processed
    
- или удаляет его
    

---

Главное преимущество:  
мы используем ACID-транзакцию одной БД вместо distributed transaction между БД и брокером.

---

Но здесь очень важно понимать:  
transactional outbox не убирает duplicates полностью.

Например:

1. relay отправил событие в Kafka
    
2. упал до mark-as-sent
    
3. после рестарта отправил снова
    

Поэтому downstream consumer всё равно должен быть идемпотентным.

---

Есть два популярных способа реализации.

---

Первый — polling publisher.

Отдельный worker:

```sql
SELECT * FROM outbox
WHERE processed = false;
```

Плюсы:

- просто
    

Минусы:

- polling overhead
    
- latency
    

---

Второй — CDC (Change Data Capture).

Например через:

- Debezium
    

Debezium читает WAL/binlog и автоматически публикует изменения outbox table в Kafka.

Это более production-grade подход.

---

Очень важно сказать, когда этот паттерн нужен.

Transactional outbox почти обязателен в:

- event-driven architecture
    
- microservices
    
- saga orchestration
    

где события являются частью бизнес-консистентности.

---

И хороший финальный тезис:

Transactional outbox — это способ атомарно сохранить бизнес-изменения и событие в одной БД-транзакции, чтобы избежать рассинхронизации между базой и message broker’ом.