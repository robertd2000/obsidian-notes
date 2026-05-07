На собеседовании под “паттернами AMQP” обычно имеют в виду не сам протокол, а типовые схемы маршрутизации и доставки сообщений, которые используются в RabbitMQ и других AMQP-брокерах.

Я бы начал с того, что в AMQP producer обычно не отправляет сообщение напрямую в queue.  
Он отправляет его в **exchange**, а exchange уже решает, куда маршрутизировать сообщение.

И именно тип exchange определяет messaging pattern.

---

Самый базовый паттерн — **Point-to-Point / Work Queue**.

Есть:

- одна queue
    
- несколько workers
    

Каждое сообщение обрабатывается только одним consumer’ом.

Это классический сценарий:

- background jobs
    
- image processing
    
- email sending
    

RabbitMQ распределяет сообщения между workers’ами, обычно round-robin с учётом prefetch.

---

Следующий — **Publish/Subscribe (fanout)**.

Producer публикует событие в fanout exchange, и сообщение копируется во все подписанные очереди.

Например:

- user_registered
    
- analytics service
    
- notification service
    
- audit service
    

Все получают одно и то же событие независимо.

Это уже event-driven architecture.

---

Дальше — **Topic routing**.

Это один из самых популярных паттернов в AMQP.

Routing key имеет структуру вроде:

```id="6s4p0v"
order.created
user.updated
payment.failed
```

А queues подписываются через wildcard patterns:

```id="r7ccvr"
order.*
*.failed
```

Это позволяет гибко маршрутизировать события.

Например:

- одна queue получает только order events
    
- другая — только failures
    

---

Есть ещё **Direct routing**.

Здесь exchange отправляет сообщение только в queue с точным matching routing key.

Это более простая и предсказуемая маршрутизация.

---

Отдельный важный паттерн — **RPC over AMQP**.

Consumer работает как remote service:

- client отправляет request message
    
- server обрабатывает
    
- отвечает в reply queue
    

Для этого используют:

- correlation_id
    
- reply_to
    

Но обычно на собесе полезно сказать, что RabbitMQ RPC — это скорее workaround, и для synchronous communication чаще лучше:

- HTTP
    
- gRPC
    

---

Ещё важный production pattern — **Dead Letter Queue (DLQ)**.

Если сообщение:

- не обработалось
    
- rejected
    
- expired
    

оно отправляется в отдельную очередь.

Это позволяет:

- не терять сообщения
    
- отдельно разбирать poison messages
    

---

Есть ещё retry patterns.

Обычно:

- consumer падает
    
- сообщение перекидывается в retry queue
    
- через TTL возвращается обратно
    

Так делают delayed retries.

---

Ещё сильный момент — competing consumers.

Несколько consumers читают одну queue:

- нагрузка распределяется
    
- сообщение получает только один worker
    

Это фундаментальная модель RabbitMQ.

---

Если всё обобщить:

AMQP patterns — это разные способы:

- маршрутизации
    
- fanout delivery
    
- load balancing
    
- retry/recovery
    

через exchanges, queues и routing keys.

---

И хороший финальный тезис:

RabbitMQ силён именно в сложной маршрутизации и delivery patterns, тогда как Kafka больше про distributed log и event streaming.

---

## ⚠️ Где валятся

- думают, что producer пишет прямо в queue
    
- не знают роль exchange
    
- путают topic exchange и Kafka topic
    
- не понимают DLQ/retry patterns
    

---

## 📚 Источники

- OASIS
    
- VMware docs
    
- RabbitMQ in Action