
https://www.youtube.com/watch?v=-8JlOr3Z0eA



# 1. Зачем закрывать тело запроса

## Основная причина: **Освобождение ресурсов**

Тело HTTP запроса (`http.Request.Body`) — это `io.ReadCloser`, который держит открытые ресурсы (сетевые соединения, файловые дескрипторы и т.д.).

## Что происходит если не закрывать

### **1. Утечка ресурсов**
```go
func badHandler(w http.ResponseWriter, r *http.Request) {
    data, _ := io.ReadAll(r.Body)
    // ЗАБЫЛИ: r.Body.Close()
    // Соединение не возвращается в pool, файловый дескриптор не освобождается
}
```

### **2. Исчерпание лимитов**
```go
// После множества таких запросов:
// - Закончатся файловые дескрипторы
// - Исчерпается pool соединений
// - Приложение упадёт или зависнет
```

## Правильные способы закрытия

### **1. Явное закрытие**
```go
func goodHandler(w http.Response.ResponseWriter, r *http.Request) {
    defer r.Body.Close()  // Гарантированное закрытие
    
    data, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Обработка data...
}
```

### **2. Через io.Copy с NopCloser**
```go
func proxyHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()
    
    // Если нужно переслать тело дальше
    resp, err := http.Post("http://backend/api", r.Header.Get("Content-Type"), r.Body)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    defer resp.Body.Close()
}
```

### **3. При частичном чтении**
```go
func partialReadHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()
    
    // Читаем только первые N байт
    var buf [1024]byte
    n, _ := r.Body.Read(buf[:])
    
    // Тело всё равно нужно закрыть, даже если прочитали не всё!
}
```

## Особые случаи

### **Пустое тело**
```go
func emptyBodyHandler(w http.ResponseWriter, r *http.Request) {
    // Даже если тело пустое - лучше закрыть
    if r.Body != nil {
        defer r.Body.Close()
    }
    
    // Обработка запроса...
}
```

### **Обработка ошибок**
```go
func errorProneHandler(w http.ResponseWriter, r *http.Request) {
    var data RequestData
    err := json.NewDecoder(r.Body).Decode(&data)
    
    // Закрываем в любом случае - даже при ошибке парсинга
    if closeErr := r.Body.Close(); closeErr != nil {
        log.Printf("Ошибка закрытия тела: %v", closeErr)
    }
    
    if err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Обработка data...
}
```

## Что происходит при закрытии

### **Освобождаются ресурсы:**
- **Сетевые соединения** возвращаются в pool
- **Файловые дескрипторы** освобождаются
- **Буферы памяти** очищаются

### **Без закрытия:**
```go
// Плохой код - утечка на каждый запрос
for i := 0; i < 10000; i++ {
    go func() {
        resp, _ := http.Get("http://example.com")
        // Забыли resp.Body.Close()
        // Через N запросов: too many open files
    }()
}
```

## Исключения (когда можно не закрывать)

### **1. HTTP/2 Server Push**
```go
// В некоторых случаях с Server Push тело может не требовать закрытия
// но лучше всегда использовать defer close для надёжности
```

### **2. Hijacked соединения**
```go
func websocketHandler(w http.ResponseWriter, r *http.Request) {
    hijacker, _ := w.(http.Hijacker)
    conn, _, _ := hijacker.Hijack()
    defer conn.Close()
    
    // После Hijack() Body уже не актуально
    // Но всё равно хорошая практика - закрыть
    if r.Body != nil {
        r.Body.Close()
    }
}
```

## Автоматическое закрытие в некоторых случаях

### **http.Server**
```go
server := &http.Server{
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,  // Автоматически закрывает idle соединения
}
```

Но это **не отменяет** необходимость явного закрытия Body!

## Best Practices

### **1. Всегда используйте defer**
```go
func bestPracticeHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()  // СРАЗУ после получения Request
    
    // Остальной код...
}
```

### **2. Проверяйте на nil**
```go
func safeHandler(w http.ResponseWriter, r *http.Request) {
    if r.Body != nil {
        defer r.Body.Close()
    }
    // Особенно важно в тестах или с кастомными Request
}
```

### **3. Логируйте ошибки закрытия**
```go
func verboseHandler(w http.ResponseWriter, r *http.Request) {
    defer func() {
        if err := r.Body.Close(); err != nil {
            log.Printf("Body close error: %v", err)
        }
    }()
}
```

## Итог

**Закрывайте тело запроса ВСЕГДА** потому что:

- ✅ **Предотвращает утечки ресурсов**
- ✅ **Избегает исчерпания файловых дескрипторов**
- ✅ **Освобождает сетевые соединения**
- ✅ **Делает приложение стабильным под нагрузкой**

`defer r.Body.Close()` — это must-have в каждом HTTP обработчике.
# 2. Когда использовать буферизированный канал

## Основные сценарии использования

### **1. Асинхронная коммуникация**
```go
// Производитель может работать быстрее потребителя
ch := make(chan int, 100) // Буфер на 100 элементов

// Производитель не блокируется пока есть место в буфере
go producer(ch) 
go consumer(ch)
```

### **2. Ограничение скорости (Rate Limiting)**
```go
// Ограничиваем количество одновременных задач
semaphore := make(chan struct{}, 10) // Не больше 10 горутин одновременно

for i := 0; i < 100; i++ {
    semaphore <- struct{}{} // Занимаем слот
    go func(task int) {
        defer func() { <-semaphore }() // Освобождаем слот
        processTask(task)
    }(i)
}
```

### **3. Балансировка нагрузки**
```go
// Распределение работы между воркерами
jobs := make(chan Job, 50)
results := make(chan Result, 50)

// Запускаем пул воркеров
for i := 0; i < 5; i++ {
    go worker(jobs, results)
}

// Отправляем задания - не блокируемся на отправке
for _, job := range jobList {
    jobs <- job
}
close(jobs)
```

### **4. Кэширование частых операций**
```go
// Пул соединений с базой данных
type ConnPool struct {
    connections chan *sql.DB
}

func NewPool(size int) *ConnPool {
    return &ConnPool{
        connections: make(chan *sql.DB, size),
    }
}

func (p *ConnPool) Get() *sql.DB {
    select {
    case conn := <-p.connections:
        return conn
    default:
        return createNewConnection()
    }
}

func (p *ConnPool) Put(conn *sql.DB) {
    select {
    case p.connections <- conn: // Возвращаем в пул если есть место
    default:
        conn.Close() // Закрываем если пул полон
    }
}
```

### **5. Обработка burst-трафика**
```go
// Сервис должен выдерживать всплески нагрузки
requests := make(chan *Request, 1000)

func handleBurst() {
    for i := 0; i < 5000; i++ {
        // Первые 1000 запросов примутся без блокировки
        requests <- &Request{ID: i}
    }
}
```

## Когда НЕ использовать буферизированные каналы

### **1. Синхронная коммуникация**
```go
// Нужна гарантия, что получатель готов
ch := make(chan int) // Небуферизированный

go func() {
    result := heavyCalculation()
    ch <- result // Блокируется пока получатель не готов
}()

value := <-ch // Гарантированно получим результат
```

### **2. Сигнальные каналы**
```go
// Только для сигналов - данные не важны
done := make(chan struct{}) // Небуферизированный

go func() {
    <-done // Чистый сигнал
    cleanup()
}()
```

## Практические примеры

### **Worker Pool с буферизированной очередью**
```go
func StartWorkerPool(workerCount, queueSize int) {
    jobs := make(chan Job, queueSize)
    results := make(chan Result, queueSize)
    
    // Воркеры
    for i := 0; i < workerCount; i++ {
        go func(id int) {
            for job := range jobs {
                results <- processJob(job)
            }
        }(i)
    }
    
    // Диспетчер заданий
    go func() {
        for {
            select {
            case job := <-jobSource:
                jobs <- job // Не блокируется пока есть место
            case <-stop:
                close(jobs)
                return
            }
        }
    }()
}
```

### **Rate Limiter**
```go
type RateLimiter struct {
    tokens chan time.Time
}

func NewRateLimiter(rate int, per time.Duration) *RateLimiter {
    rl := &RateLimiter{
        tokens: make(chan time.Time, rate),
    }
    
    // Наполняем токены
    go func() {
        ticker := time.NewTicker(per / time.Duration(rate))
        defer ticker.Stop()
        
        for t := range ticker.C {
            select {
            case rl.tokens <- t:
            default: // Пропускаем если буфер полон
            }
        }
    }()
    
    return rl
}

func (rl *RateLimiter) Allow() bool {
    select {
    case <-rl.tokens:
        return true
    default:
        return false
    }
}
```

### **Обработка событий с приоритетом**
```go
type EventProcessor struct {
    highPriority chan Event
    lowPriority  chan Event
}

func NewEventProcessor() *EventProcessor {
    return &EventProcessor{
        highPriority: make(chan Event, 10),  // Маленький буфер для срочных
        lowPriority:  make(chan Event, 100), // Большой буфер для фоновых
    }
}

func (ep *EventProcessor) Process() {
    for {
        select {
        case event := <-ep.highPriority:
            processHighPriority(event) // Обрабатываем в первую очередь
        default:
            select {
            case event := <-ep.highPriority:
                processHighPriority(event)
            case event := <-ep.lowPriority:
                processLowPriority(event) // Фоновая обработка
            }
        }
    }
}
```

## Правила выбора размера буфера

### **Маленький буфер (1-10)**
```go
// Для синхронизации горутин
syncCh := make(chan struct{}, 1)

// Для ограничения concurrency
limiter := make(chan struct{}, 5)
```

### **Средний буфер (10-100)**
```go
// Для балансировки производитель-потребитель
jobs := make(chan Task, 50)

// Для обработки умеренных всплесков
events := make(chan Event, 100)
```

### **Большой буфер (100+)**
```go
// Для кэширования или очень bursty workload
cache := make(chan CacheEntry, 1000)

// Когда потребитель может сильно отставать
logs := make(chan LogEntry, 10000)
```

## Опасности буферизированных каналов

### **1. Маскировка проблем**
```go
// С буфером deadlock может проявиться позже
ch := make(chan int, 1000)
ch <- 1
ch <- 2
// ... много операций
// Deadlock проявится только когда буфер заполнится
```

### **2. Потребление памяти**
```go
// Большой буфер = много памяти
bigChannel := make(chan []byte, 10000)
// 10,000 * sizeof([]byte) = значительное потребление
```

### **3. Потеря данных**
```go
select {
case ch <- data: // Может не отправиться если буфер полон
default:
    // Данные теряются!
    log.Println("Channel full, data lost")
}
```

## Best Practices

### **1. Начинайте с небуферизированных**
```go
// Сначала сделайте правильную синхронизацию
ch := make(chan int)
// Потом добавляйте буфер только если нужно
```

### **2. Измеряйте и настраивайте**
```go
// Мониторинг заполненности буфера
func monitorChannel(ch chan int) {
    go func() {
        for {
            utilization := float64(len(ch)) / float64(cap(ch)) * 100
            metrics.ChannelUtilization.Set(utilization)
            time.Sleep(time.Second)
        }
    }()
}
```

### **3. Документируйте семантику**
```go
// С буфером 0 - синхронная коммуникация
var SyncChannel chan Message // Отправитель ждёт получателя

// С буфером > 0 - асинхронная коммуникация  
var AsyncChannel chan Message // Отправитель не блокируется
```

## Итог

**Используйте буферизированные каналы когда:**
- ✅ Нужна асинхронная коммуникация
- ✅ Производитель и потребитель работают в разном темпе
- ✅ Нужно обрабатывать всплески нагрузки
- ✅ Реализуете пулы или ограничители

**Избегайте когда:**
- ⚠️ Нужна строгая синхронизация
- ⚠️ Важна гарантия доставки
- ⚠️ Ресурсы ограничены

Размер буфера выбирайте на основе измерений и понимания workload'а вашего приложения.
# 3. awasp top 10
# 4. sql injections
# 5. loot box / tranactional loot box
# 6. join sql
# 7. left/right/inner join sql
# 8. sql index
# 9. виды индексов
# 10. авто индексы
# 11. sql explain
# 12. партиционыирование sql


[  
00:27](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=27s) – Начало работы Уверенность: Средняя. Правильность: Подключил необходимые пакеты, но были небольшие задержки. [01:43](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=103s) – Обработка ошибок Уверенность: Средняя. Правильность: Верно обработал ошибки, но не сразу учёл все нюансы. [05:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=319s) – Закрытие тела запроса Уверенность: Средняя. Правильность: Пропустил необходимость закрытия тела запроса, но потом исправил. [06:28](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=388s) – Использование defer Уверенность: Средняя. Правильность: Правильно применил defer, но не сразу учёл все кейсы. [08:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=499s) – Модификация программы Уверенность: Средняя. Правильность: Модифицировал программу для работы с горутинами, но были ошибки. [13:03](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=783s) – Работа с каналами Уверенность: Средняя. Правильность: Выбрал небуферизированные каналы, объяснил выбор. [15:09](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=909s) – Обработка неизвестного количества данных Уверенность: Средняя. Правильность: Предложил решение для обработки данных из внешнего источника. [18:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1099s) – Параллельное выполнение Уверенность: Средняя. Правильность: Реализовал воркер-пул, но были недочёты в обработке контекста. [26:37](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1597s) – Код-ревью Уверенность: Высокая Правильность: Нашёл все основные ошибки [31:45](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1905s) – Хранение данных Уверенность: Высокая Правильность: Обсудил хранение паролей, предложил использование секретов. [33:03](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1983s) – Обсуждение структуры данных Уверенность: Высокая Правильность: Предложил вынести структуру в отдельный файл. [34:40](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2080s) – Поиск оптимального решения Уверенность: Средняя. Правильность: Обсудил архитектурные подходы, но не сразу предложил паттерн Outbox [36:24](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2184s) – Реализация стратегии Уверенность: Средняя. Правильность: Планировал использование стратегии, но не реализовал. [37:01](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2221s) – Анализ кода и обработка ошибок Уверенность: Высокая Правильность: Обнаружил некорректное использование токена. [39:42](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2382s) – Проверка подключения к базе Уверенность: Средняя. Правильность: Обсудил критичность подключения, но не предложил решение. [40:49](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2449s) – Избыточная обработка ошибок Уверенность: Средняя. Правильность: Обнаружил избыточность, но не исправил. [42:05](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2525s) – Проблемы с построением запросов Уверенность: Средняя. Правильность: Указал на риски инъекций, но не предложил решение. [43:21](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2601s) – Логика работы с соединениями Уверенность: Средняя. Правильность: Отметил неэффективность, предложил пул соединений. [45:47](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2747s) – Архитектура системы Уверенность: Средняя. Правильность: Предложил Kafka, но не учёл ограничения. [48:07](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2887s) – Синхронное взаимодействие с аналитикой Уверенность: Средняя. Правильность: Обсудил синхронную обработку, но не предложил решение. [52:33](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3153s) – Последовательность действий Уверенность: Средняя. Правильность: Описал последовательность, но не учёл все нюансы. [55:30](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3330s) – Управление нагрузкой Уверенность: Выше среднего. Правильность: Предложил увеличение воркеров, но не детализировал. [57:49](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3469s) – Создание таблиц Уверенность: Средняя. Правильность: Создал таблицы, но были ошибки в полях. [01:09:38](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4178s) – Тестирование запросов Уверенность: Средняя. Правильность: Протестировал запросы, но не все изменения. [01:10:45](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4245s) – Введение в индексы Уверенность: Высокая Правильность: Корректно описал индексы. [01:11:34](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4294s) – Применение индексов Уверенность: Средняя. Правильность: Правильно объяснил использование индексов. [01:12:36](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4356s) – Автоматические индексы Уверенность: Высокая. Правильность: Упомянул B-Tree индексы. [01:13:07](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4387s) – Анализ запросов Уверенность: Выше среднего. Правильность: Описал инструмент EXPLAIN, но не все метрики. [01:15:09](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4509s) – Оптимизация запросов Уверенность: Выше среднего. Правильность: Предложил партиционирование, но не детализировал. [01:17:41](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4661s) – Репликация Уверенность: Средняя. Правильность: Обсудил репликацию, но не учёл частую запись.

##