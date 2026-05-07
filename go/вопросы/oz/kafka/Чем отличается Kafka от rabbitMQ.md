На собеседовании тут обычно проверяют не “знаешь ли ты две технологии”, а понимаешь ли ты **разницу моделей**.

Я бы отвечал так:

Apache Kafka и RabbitMQ обе решают задачу передачи сообщений между сервисами, но делают это принципиально по-разному.

RabbitMQ — это классический message broker / queue system.  
Kafka — это distributed append-only log / event streaming platform.

И из этой разницы вытекает почти всё остальное.

---

В RabbitMQ сообщение обычно живёт по модели:

- producer отправил
    
- broker доставил consumer’у
    
- consumer ack’нул
    
- сообщение удалилось из очереди
    

То есть это именно queue semantics.

В Kafka сообщение не удаляется после чтения. Оно сохраняется в log определённое время (retention), а consumer хранит только offset — место, до которого дочитал.

Поэтому Kafka позволяет:

- перечитывать сообщения
    
- делать replay
    
- подключать новых consumers к старым данным
    

RabbitMQ для такого подходит намного хуже.

---

Следующее важное отличие — throughput и модель хранения.

Kafka оптимизирована под:

- sequential disk writes
    
- batching
    
- append-only log
    

Поэтому она очень хорошо работает с огромным потоком событий:

- миллионы сообщений
    
- event streaming
    
- analytics
    
- telemetry
    

RabbitMQ обычно лучше подходит для:

- task queues
    
- RPC
    
- сложной маршрутизации сообщений
    
- low-latency message delivery
    

---

Дальше consumer model.

В Kafka consumer pull’ит сообщения сам.  
В RabbitMQ broker активно доставляет сообщения consumer’у.

Из-за этого Kafka лучше контролирует:

- backpressure
    
- replay
    
- независимость consumers
    

---

Следующее — ordering.

В Kafka порядок гарантируется внутри partition.

В RabbitMQ порядок зависит от очереди и конкуренции consumers, и при параллельной обработке поддерживать strict ordering сложнее.

---

Теперь routing.

RabbitMQ очень сильна в routing logic:

- exchanges
    
- topics
    
- fanout
    
- direct routing
    
- headers
    

Kafka исторически проще:

- topic + partition
    
- key-based routing
    

Kafka — про streaming, RabbitMQ — про routing/message delivery.

---

Ещё важная разница — масштабирование.

Kafka изначально проектировалась как distributed system:

- partitioning
    
- replication
    
- horizontal scaling
    

RabbitMQ масштабировать сложнее при очень больших нагрузках.

---

Теперь про delivery semantics.

Обе системы могут поддерживать:

- at-most-once
    
- at-least-once
    

Kafka ещё хорошо интегрирована с exactly-once semantics через idempotent producer и transactions.

---

Если говорить практично:

RabbitMQ я бы выбрал для:

- фоновых задач
    
- workflow
    
- RPC между сервисами
    
- сложной маршрутизации
    

Kafka:

- event-driven architecture
    
- event sourcing
    
- analytics pipelines
    
- stream processing
    
- high throughput systems
    

---

И хороший финальный тезис:

RabbitMQ — это прежде всего message broker для доставки и маршрутизации сообщений,  
а Kafka — распределённый лог событий для масштабируемого стриминга и хранения event history.

---

## ⚠️ Где валятся

- говорят “Kafka быстрее RabbitMQ” без объяснения почему
    
- не понимают log-based модель Kafka
    
- думают, что Kafka — просто “очередь”
    
- не различают push vs pull model
    

---

## 📚 Источники

- Apache Software Foundation docs
    
- VMware docs
    
- Designing Data-Intensive Applications