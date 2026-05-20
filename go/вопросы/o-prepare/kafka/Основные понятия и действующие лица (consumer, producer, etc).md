На собеседовании по Apache Kafka обычно важно не просто перечислить термины, а показать понимание того, как поток данных проходит через систему.

Я бы объяснял так:

Kafka — это распределённый commit log / event streaming system.  
Продюсеры пишут события в топики, Kafka хранит их как упорядоченный лог, а консюмеры читают эти события независимо друг от друга.

Ключевая идея Kafka — данные не “пушатся” потребителям, как в классических очередях, а хранятся в логе, и consumer сам решает, что и когда читать.

---

Главные сущности:

**Producer** — сервис, который отправляет сообщения в Kafka.  
Он пишет данные в topic и может выбирать partition либо явно, либо через key hashing.

Важно понимать, что producer взаимодействует не с “очередью”, а именно с append-only log. Сообщения в partition всегда добавляются в конец.

---

**Topic** — это логическая категория сообщений.  
Например:

- `orders`
    
- `payments`
    
- `user_events`
    

Но физически topic делится на partitions.

---

**Partition** — самая важная сущность в Kafka.

Partition — это:

- упорядоченный immutable log
    
- последовательность сообщений с offset
    

Порядок гарантируется только внутри partition.

Именно partition — единица:

- масштабирования
    
- параллелизма
    
- репликации
    

---

**Offset** — позиция сообщения внутри partition.

Kafka не удаляет сообщение после чтения. Вместо этого consumer хранит offset и говорит:  
“я прочитал до этого места”.

Это фундаментальное отличие от традиционных message queue.

---

**Consumer** — читатель сообщений.

Consumer pull’ит данные из Kafka и сам контролирует:

- скорость чтения
    
- offset
    
- retry
    

---

**Consumer Group** — механизм масштабирования чтения.

Если несколько consumers входят в одну group:

- каждая partition читается только одним consumer внутри группы
    
- нагрузка распределяется автоматически
    

Например:

- 6 partitions
    
- 3 consumers
    

→ каждый читает по 2 partition.

---

Очень важно понимать rebalance.

Когда:

- consumer упал
    
- появился новый
    
- изменилась topology
    

Kafka перераспределяет partitions между consumers.

Во время rebalance возможны:

- stop-the-world паузы
    
- duplicate processing
    

---

Дальше — broker.

**Broker** — сервер Kafka.

Cluster состоит из нескольких brokers, и partitions распределены между ними.

---

Каждая partition имеет:

- leader
    
- followers
    

Producer и consumer работают через leader.

Followers реплицируют данные для fault tolerance.

---

Ещё важное понятие — replication factor.

Если RF=3:

- partition хранится на трёх brokers
    
- один leader
    
- два replicas
    

Это нужно для отказоустойчивости.

---

Отдельно полезно упомянуть ISR (in-sync replicas).

Это реплики, которые успевают за leader.  
Kafka считает запись надёжной только если данные попали в ISR (в зависимости от acks/min.insync.replicas).

---

Если коротко обобщить архитектуру:

Producer пишет события → Kafka хранит их как partitioned replicated log → Consumers независимо читают сообщения по offset.

---

И хороший финальный тезис:

Kafka — это не просто очередь сообщений, а распределённый отказоустойчивый лог событий, где ключевые вещи — partitioning, replication и consumer groups.

---

## ⚠️ Где валятся

- думают, что Kafka удаляет сообщение после чтения
    
- не понимают consumer groups
    
- путают topic и partition
    
- думают, что порядок глобальный, а не внутри partition
    

---

## 📚 Источники

- Apache Software Foundation docs
    
- Designing Data-Intensive Applications
    
- Kafka: The Definitive Guide
Если бы на собеседовании меня попросили объяснить основные сущности Apache Kafka, я бы начал не со списка терминов, а с общей модели.

Kafka — это распределённый append-only log для передачи событий между сервисами. Один сервис пишет события, другие их читают. При этом Kafka хранит сообщения у себя и не удаляет их после чтения, как классическая очередь.

Дальше уже можно разложить роли.

Producer — это сервис, который отправляет сообщения в Kafka. Например, сервис заказов создал order и отправил событие `order_created`. Producer пишет сообщение не “в Kafka целиком”, а в конкретный topic и partition.

Topic — это логическая категория сообщений. Например:

- orders
    
- payments
    
- notifications
    

Но физически topic разбит на partitions. И partition — это ключевая сущность Kafka. По сути это просто упорядоченный лог сообщений. У каждого сообщения внутри partition есть offset — номер позиции.

Важно понимать, что порядок в Kafka гарантируется только внутри partition. Глобального порядка между всеми partition нет.

Дальше producer при отправке определяет, в какую partition попадёт сообщение. Обычно это делают через key. Например, все события одного пользователя отправляются с одинаковым key, чтобы они всегда попадали в одну partition и сохранялся порядок.

Consumer — это сервис, который читает сообщения из Kafka. В отличие от обычной очереди Kafka не “пушит” сообщения. Consumer сам читает их и хранит offset — то есть место, до которого он дочитал лог.

Это важный момент: Kafka хранит сообщения независимо от consumer’ов. Поэтому одно и то же сообщение могут читать несколько разных систем независимо друг от друга.

Например:

- analytics service
    
- notification service
    
- recommendation service
    

Все могут читать один topic параллельно.

Дальше появляется consumer group. Это механизм масштабирования.

Если consumers находятся в одной группе, Kafka распределяет partitions между ними. Одну partition внутри группы читает только один consumer.

Например:

- topic имеет 6 partitions
    
- consumer group имеет 3 consumers
    

Тогда каждый consumer получит примерно по 2 partitions.

Это даёт горизонтальное масштабирование.

Но тут есть важное ограничение:  
количество активно работающих consumers в group не может быть больше количества partitions. Иначе часть consumers будет простаивать.

Теперь про brokers.

Broker — это сервер Kafka. Kafka cluster состоит из нескольких brokers, а partitions распределяются между ними.

Для отказоустойчивости каждая partition реплицируется. У partition есть:

- leader
    
- follower replicas
    

Все записи и чтения идут через leader. Followers копируют данные.

Если broker с leader падает — один из followers становится новым leader.

Здесь важно упомянуть replication factor и ISR.

Replication factor определяет, сколько копий partition хранится в кластере.

ISR — это реплики, которые успевают синхронизироваться с leader. Kafka использует ISR для гарантий надёжности записи.

И вот здесь уже можно красиво закончить:

Kafka — это распределённый реплицируемый лог, где:

- producer пишет события,
    
- broker’ы хранят их в partition,
    
- consumer’ы читают их по offset,
    
- а consumer groups позволяют масштабировать обработку.

Вот это уже хороший “инженерный” вопрос, потому что тут важно показать, что ты понимаешь trade-offs, а не просто знаешь определения.

Я бы отвечал так.

Когда выбираешь параметры в Apache Kafka, нужно смотреть на:

- надёжность
    
- throughput
    
- latency
    
- стоимость
    
- сценарий отказов
    

Потому что почти все настройки Kafka — это компромисс.

---

Начнём с replication factor.

Replication factor определяет, сколько копий partition хранится в кластере.

Например:

- RF=1 → копий нет
    
- RF=3 → leader + две replicas
    

На практике для продакшна почти всегда выбирают RF=3.

Почему:

- один broker может упасть
    
- Kafka всё ещё сохранит данные
    
- и сможет выбрать нового leader
    

RF=1 обычно только для dev/test, потому что потеря broker = потеря данных.

Дальше важно понимать, что высокий replication factor:

- увеличивает надёжность
    
- но увеличивает:
    
    - сетевой трафик
        
    - нагрузку на диск
        
    - latency записи
        

Потому что leader должен реплицировать данные followers.

---

Следующий важный параметр — количество partitions.

Partition — это единица параллелизма.

Больше partitions:

- выше throughput
    
- больше consumers можно масштабировать
    
- лучше parallel processing
    

Но есть цена:

- больше metadata
    
- сложнее rebalance
    
- больше open file handles
    
- больше network overhead
    

Типичная ошибка — делать тысячи partitions “на всякий случай”.

---

Как обычно выбирают partitions?

Смотрят на:

- expected throughput
    
- количество consumers
    
- key distribution
    

Например:  
если нужно максимум 10 параллельных consumers:  
→ partitions должно быть минимум 10.

Но делать 1000 partitions ради 10 consumers — плохая идея.

---

Теперь consumer group.

Consumer group — это способ масштабировать обработку.

Главное правило:

- одна partition внутри group читается только одним consumer.
    

Поэтому:

- consumers > partitions → часть consumers idle
    
- partitions > consumers → consumers читают несколько partitions
    

Обычно количество consumers выбирают:

- по CPU-bound / IO-bound обработке
    
- по throughput
    
- по lag
    

---

Теперь важный момент — acks.

Producer может писать с разными гарантиями:

`acks=0`  
→ максимальная скорость  
→ можно потерять данные

`acks=1`  
→ leader подтвердил запись  
→ если leader упал до replication — возможна потеря

`acks=all`  
→ запись подтверждена ISR replicas  
→ safest option

Для критичных данных обычно:

```id="o0g8xg"
acks=all
min.insync.replicas=2
RF=3
```

Это позволяет пережить падение одного broker без потери данных.

---

Дальше retention.

Kafka хранит сообщения даже после чтения.

Retention выбирают исходя из:

- replay needs
    
- storage cost
    
- recovery strategy
    

Например:

- event sourcing → долгий retention
    
- transient events → короткий
    

---

Ещё сильный момент — key design.

Если key распределён плохо:

- одна partition становится hotspot
    
- появляется skew
    

Например:  
если все события пишутся с одним key:  
→ весь traffic идёт в одну partition  
→ теряется масштабирование

---

И хороший финальный тезис:

Kafka настраивается не “по best practices”, а под конкретный workload.

Replication factor — про отказоустойчивость,  
partitions — про параллелизм,  
consumer groups — про масштабирование обработки,  
acks и ISR — про гарантии доставки.

И все эти параметры напрямую влияют друг на друга.