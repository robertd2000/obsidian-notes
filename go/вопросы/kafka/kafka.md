Отличные вопросы! Вот развернутые ответы на них:

---

## 4. Что такое отказоустойчивость Kafka и за счёт чего она достигается?

**Отказоустойчивость Kafka** — это способность системы продолжать работу при сбоях отдельных компонентов без потери данных и с минимальным простоем.

**Достигается за счет:**

### **Репликация данных**
- Каждая партиция реплицируется на несколько брокеров (replication factor)
- Одна реплика — лидер (leader), остальные — последователи (followers)
- Лидер обрабатывает все запросы чтения/записи
- При падении лидера один из followers автоматически становится новым лидером

### **Распределенная архитектура**
```bash
# Пример топика с replication factor = 3
Topic: orders
Partition 0: [Broker1(leader), Broker2(follower), Broker3(follower)]
Partition 1: [Broker2(leader), Broker3(follower), Broker1(follower)]
```

### **Механизмы обеспечения отказоустойчивости:**
- **ISR (In-Sync Replicas)** — синхронизированные реплики
- **Controller** — специальный брокер, управляющий распределением партиций
- **ZooKeeper/KRaft** — координация и хранение метаданных
- **Acks** — подтверждения записи от реплик

**Пример настройки в Go:**
```go
writer := kafka.NewWriter(kafka.WriterConfig{
    Brokers: []string{"broker1:9092", "broker2:9092", "broker3:9092"},
    Topic:   "orders",
    Balancer: &kafka.Hash{},
    RequiredAcks: kafka.RequireAll, // Ждем подтверждения от всех реплик
})
```

---

## 5. Какие гарантии доставки предоставляет Kafka?

Kafka предоставляет настраиваемые гарантии доставки:

### **Producer гарантии:**
- **Idempotent Producer** — гарантирует exactly-once семантику в пределах одной сессии
- **Transactions** — атомарная запись в несколько партиций/топиков
- **Acks** — контроль подтверждения записи

### **Consumer гарантии:**
- **Read committed** — чтение только закоммиченных сообщений
- **Consumer groups** — отслеживание прогресса обработки
- **Offset management** — контроль позиции чтения

---

## 6. Чем отличаются `at least once`, `at most once` и `exactly once`?

### **At Most Once**
- Сообщение может быть потеряно, но никогда не доставлено дважды
- **Producer**: отправляет без ожидания подтверждения
- **Consumer**: коммитит offset до обработки сообщения
- **Использование**: когда потери допустимы (метрики, логи)

```go
// Producer - no acks
writer := kafka.Writer{
    RequiredAcks: kafka.RequireNone,
}

// Consumer - commit before processing
msg, _ := reader.ReadMessage(ctx)
reader.CommitMessages(ctx, msg) // Коммит ДО обработки
process(msg)                    // Может упасть - сообщение потеряно
```

### **At Least Once** ⭐ (наиболее популярен)
- Сообщение никогда не теряется, но возможны дубли
- **Producer**: ждет подтверждения от реплик
- **Consumer**: коммитит offset после успешной обработки
- **Использование**: когда дублирование допустимо (чаще всего)

```go
// Producer - wait for all replicas
writer := kafka.Writer{
    RequiredAcks: kafka.RequireAll,
}

// Consumer - commit after processing
msg, _ := reader.ReadMessage(ctx)
process(msg)                    // Обработка
reader.CommitMessages(ctx, msg) // Коммит ПОСЛЕ - если упадет здесь, сообщение прочитается повторно
```

### **Exactly Once**
- Каждое сообщение доставляется ровно один раз
- **Механизм**: транзакции + идемпотентный producer
- **Сложность**: требует дополнительной настройки
- **Использование**: финансовые операции, критические данные

```go
// Producer с транзакциями
writer := kafka.NewWriter(kafka.WriterConfig{
    Brokers: brokers,
    Topic:   topic,
    Balancer: &kafka.Hash{},
    TransactionalID: "my-transactional-producer",
})

writer.BeginTransaction()
err := writer.WriteMessages(ctx, messages...)
if err != nil {
    writer.AbortTransaction(ctx)
} else {
    writer.CommitTransaction(ctx)
}
```

---

## 7. Какие плюсы и минусы у микросервисной архитектуры?

### **Плюсы:**
- **Независимое развертывание** — можно обновлять сервисы по отдельности
- **Технологическое разнообразие** — каждый сервис на своем стеке
- **Масштабируемость** — масштабируем только нужные сервисы
- **Устойчивость к сбоям** — падение одного сервиса не убивает всю систему
- **Более простая поддержка** — меньшие кодовые базы

### **Минусы:**
- **Сложность** — распределенные транзакции, отладка
- **Накладные расходы** — сетевые задержки, сериализация
- **Data consistency** — сложно поддерживать согласованность данных
- **Operational overhead** — нужен мониторинг, orchestration
- **Тестирование** — сложнее тестировать взаимодействие сервисов

---

## 8. Как реализуются распределённые транзакции между микросервисами? Паттерн Saga

### **Проблема:**
В микросервисах нет единой БД → нельзя использовать ACID транзакции.

### **Решение: Saga Pattern**
Лонг-раннинг транзакция, состоящая из цепочки локальных транзакций.

### **Два подхода:**

#### **Хореография (Choreography)**
- Каждый сервис знает только о следующем шаге
- События публикуются в брокер (Kafka)
- **Плюсы**: слабая связанность
- **Минусы**: сложно отслеживать состояние

```go
// Пример: создание заказа
OrderService → "OrderCreated" → PaymentService → "PaymentProcessed" → InventoryService
```

#### **Оркестрация (Orchestration)**
- Есть центральный координатор (orchestrator)
- Координатор управляет потоком выполнения
- **Плюсы**: проще отслеживать, явный контроль
- **Минусы**: точка отказа

```go
// Go пример оркестратора
type OrderSaga struct {
    orderService    OrderService
    paymentService  PaymentService
    inventoryService InventoryService
}

func (s *OrderSaga) CreateOrder(ctx context.Context, order Order) error {
    // 1. Создаем заказ
    if err := s.orderService.Create(order); err != nil {
        return err
    }
    
    // 2. Списание денег
    if err := s.paymentService.ProcessPayment(order); err != nil {
        s.orderService.Cancel(order.ID) // Компенсирующее действие
        return err
    }
    
    // 3. Резервирование товара
    if err := s.inventoryService.Reserve(order); err != nil {
        s.paymentService.Refund(order.ID) // Компенсирующее действие
        s.orderService.Cancel(order.ID)
        return err
    }
    
    return nil
}
```

---

## 9. Как решить проблему дублирования запросов в системе с очередями?

### **Проблема:**
При at-least-once доставке возможны дублирующиеся сообщения.

### **Решение: Идемпотентность**

#### **1. Идемпотентный Consumer**
```go
type MessageProcessor struct {
    processedIDs *cache.Cache // или Redis
}

func (p *MessageProcessor) Process(msg kafka.Message) error {
    // Проверяем, обрабатывали ли уже это сообщение
    if p.processedIDs.Exists(msg.Key) {
        return nil // Пропускаем дубль
    }
    
    // Обработка
    err := businessLogic(msg)
    if err != nil {
        return err
    }
    
    // Сохраняем ID обработанного сообщения
    p.processedIDs.Set(msg.Key, true)
    return nil
}
```

#### **2. Дедупликация по ключу сообщения**
```go
func (p *OrderProcessor) ProcessOrder(msg kafka.Message) error {
    var order Order
    json.Unmarshal(msg.Value, &order)
    
    // Проверяем в БД, не обрабатывался ли уже этот заказ
    existing, err := p.orderRepo.GetByID(order.ID)
    if err == nil && existing != nil {
        return nil // Уже обработан
    }
    
    // Создаем заказ с идемпотентным ключом
    err = p.orderRepo.CreateWithID(order.ID, order)
    if err != nil && isDuplicateError(err) {
        return nil // Конфликт - уже существует
    }
    
    return err
}
```

#### **3. Использование транзакций Kafka**
```go
// Consumer в транзакционном режиме
reader := kafka.NewReader(kafka.ReaderConfig{
    Brokers: brokers,
    Topic:   topic,
    GroupID: groupID,
    IsolationLevel: kafka.ReadCommitted, // Только закоммиченные сообщения
})
```

---

## 10. Как масштабировать микросервисы? Что происходит при падении?

### **Стратегии масштабирования:**

#### **Горизонтальное масштабирование**
```yaml
# Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 3  # 3 инстанса сервиса
  template:
    spec:
      containers:
      - name: order-service
        image: order-service:v1
```

#### **Шардирование**
```go
// Пример: шардирование по userId
func getShard(userID string) string {
    hash := sha256.Sum256([]byte(userID))
    shardIndex := int(hash[0]) % len(shards)
    return shards[shardIndex]
}
```

#### **Автомасштабирование**
```yaml
# HPA в Kubernetes
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### **Что происходит при падении сервиса:**

#### **1. Обнаружение**
- Health checks (Kubernetes liveness/readiness probes)
- Service mesh (Istio, Linkerd)
- Circuit breakers

```go
// Circuit breaker в Go
cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name: "payment-service",
    Timeout: 30 * time.Second,
    ReadyToTrip: func(counts gobreaker.Counts) bool {
        return counts.ConsecutiveFailures > 5
    },
})

result, err := cb.Execute(func() (interface{}, error) {
    return paymentClient.Process(request)
})
```

#### **2. Восстановление**
- **Kubernetes**: автоматический рестарт упавших подов
- **Service Discovery**: автоматическое обновление эндпоинтов
- **Load Balancer**: перенаправление трафика на здоровые инстансы

#### **3. Graceful Degradation**
```go
func (s *OrderService) CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error) {
    // Пытаемся списать деньги
    err := s.paymentService.ProcessPayment(req.Payment)
    if err != nil {
        // Если платежный сервис недоступен, сохраняем заказ в pending
        req.Status = "payment_pending"
        logger.Warn("Payment service unavailable, order saved as pending")
    }
    
    return s.orderRepo.Save(req)
}
```

#### **4. Retry с экспоненциальной backoff**
```go
// Retry стратегия
retryOp := retry.ExponentialBackoff{
    InitialInterval:     100 * time.Millisecond,
    RandomizationFactor: 0.5,
    Multiplier:          2,
    MaxInterval:         10 * time.Second,
    MaxElapsedTime:      30 * time.Second,
}

err := retryOp.Retry(ctx, func(ctx context.Context) error {
    return paymentService.Process(request)
})
```

Эти механизмы обеспечивают высокую доступность и отказоустойчивость микросервисной архитектуры даже при сбоях отдельных компонентов.