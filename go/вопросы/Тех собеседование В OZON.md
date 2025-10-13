
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
# 3. OWASP top 10
**OWASP** (Open Web Application Security Project) — открытый проект по безопасности веб-приложений, созданный и поддерживаемый некоммерческой организацией OWASP Foundation.

Эксперты организации каждые 3–4 года обновляют **OWASP Top Ten** — список критических уязвимостей веб-приложений. Он помогает разработчикам и специалистам по информационной безопасности создавать и поддерживать безопасные сайты и приложения.

В [последней редакции](https://owasp.org/www-project-top-ten/) OWASP Top Ten названы следующие уязвимости:

- [нарушение контроля доступа](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-1);
- [недочёты криптографии](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-2);
- [инъекции](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-3);
- [небезопасный дизайн](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-4);
- [небезопасная конфигурация](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-5);
- [использование уязвимых или устаревших компонентов](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-6);
- [ошибки идентификации и аутентификации](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-7);
- [нарушения целостности программного обеспечения и данных](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-8);
- [ошибки логирования и мониторинга безопасности](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-9);
- [подделка запросов на стороне сервера](https://skillbox.ru/media/code/owasp-top-10-samye-rasprostranyennye-uyazvimosti-vebprilozheniy/#stk-10).

Познакомимся с каждым видом уязвимостей и разберёмся, как их можно устранить.

  
  

## **Нарушение контроля доступа**

Это набор уязвимостей, при которых система плохо контролирует уровни доступа к информации или к своей функциональности. Из-за этого злоумышленники могут пользоваться функциями, к которым не должны иметь доступа.

Представьте веб-приложение, где каждая учётная запись имеет разные права доступа. Если оно слабо защищено, злоумышленник может модифицировать запросы или параметры URL, чтобы получить доступ к данным, на которые у него нет права.

Такая уязвимость может привести к разглашению или утрате конфиденциальной информации, взлому учётных записей пользователей и нарушению целостности данных.

**Что делать:**

- Проектируйте контроль доступа на основе принципа наименьших привилегий. Пользователи должны иметь только те права, которые необходимы для выполнения их задач.
- Проводите аутентификацию и авторизацию на всех уровнях приложения— и на серверной, и на клиентской стороне.
- Регулярно проводите тестирование и аудит контроля доступа.

**Чего не стоит делать:**

- Полагаться только на скрытие ссылок или кнопок в пользовательском интерфейсе для ограничения доступа. Это не предотвратит доступ к закрытой функциональности по прямым запросам.
- Доверять пользовательским входным данным при авторизации. Всегда следует проводить проверку на сервере.
- Оставлять прежней политику контроля доступа при изменении требований и бизнес-логики приложения.

## **Недостатки криптографии**

Недостатки криптографии — это уязвимости, связанные с неправильной настройкой и использованием криптографических методов для защиты данных. К ним относят недостаточную длину ключей, ненадёжные условия их хранения, использование устаревших алгоритмов и другие ошибки в криптографической реализации.

Слабая криптография — как картонный сейф: делает данные уязвимыми для атакующих, но создаёт иллюзию защищённости. Например, если веб-приложение использует устаревший и слабый алгоритм шифрования для защиты паролей пользователей, то хакер довольно быстро взломает его методом перебора. Но разработчики будут думать, что их система защищена.

**Что делать:**

- Обновляйте и пересматривайте криптографические методы и ключи с учётом последних рекомендаций и стандартов.
- Храните криптографические ключи в надёжном месте. Избегайте их хранения вместе с кодом приложения или в открытом виде.

**Чего не стоит делать:**

- Использовать устаревшие или слабые алгоритмы шифрования.
- Реализовывать собственные криптографические методы, если вы не являетесь экспертом в этой области.
- Хранить криптографические ключи в открытом виде или внутри кода приложения.

## **Инъекции**

Инъекция — это пользовательский ввод с вредоносным кодом. Чаще всего инъекции включают SQL-запросы и команды на языке оболочки операционной системы.

Инъекции позволяют злоумышленникам внедрять свой вредоносный код на сервер и выполнять его. Результат — потеря данных, кража данных или повреждение системы.

Представьте, что у компании есть база данных с информацией о клиентах. Если в форме ввода пользовательской информации — допустим, для обращения в службу поддержки — не установлена фильтрация и валидация вводимых данных, то злоумышленник может написать в ней обычный SQL-запрос и получить в ответ от сервера конфиденциальную информацию из базы клиентов.

**Что делать:**

- Используйте параметризованные запросы или ORM (object-relational mapping) для работы с базой данных.
- Валидируйте и фильтруйте входные данные. Принимайте только допустимые символы и структуры данных.
- Применяйте принцип наименьших привилегий: ограничивайте права доступа к базе данных необходимыми.
- Используйте LIMIT и другие элементы управления SQL в запросах для предотвращения массового раскрытия записей в случае SQL-инъекции.

**Чего не стоит делать:**

- Конкатенировать и вставлять непроверенные данные пользователя напрямую в SQL-запросы, команды операционной системы или другие исполняемые на сервере контексты.
- Надеяться на то, что фильтрация одного типа данных предотвратит инъекции. Злоумышленники могут использовать разные методы атак.
- Хранить конфиденциальные данные в чистом тексте без шифрования в базе данных.

![](https://skillbox.ru/upload/setka_images/12352812102023_8f981a9c531888cb7306f4b47ac1979cdf88759d.JPG)

Читайте также:

[Устраняем уязвимости: как защитить сайт от SQL-инъекции](https://skillbox.ru/media/code/kak_zashchitit_sayt_ot_sql_inektsii/?utm_source=media&utm_medium=link&utm_campaign=all_all_media_links_links_articles_all_all_skillbox)

## **Небезопасный дизайн архитектуры приложения**

Широкая категория уязвимостей, впервые появившаяся в последней версии OWASP Top Ten. Уязвимости этой категории возникают потому, что сама логика работы приложения может позволять использовать существующие функции для взлома.

Например, в веб-приложение пользователи загружают файлы на сервер без их проверки. Злоумышленники могут использовать эту функцию и загрузить на сервер исполняемый файл с вредоносным кодом.

**Что делать:**

- Продумывайте аспекты безопасности на ранних этапах проектирования приложения.
- Оценивайте потенциальные угрозы и риски на этапе проектирования и разрабатывайте меры их предотвращения.
- Обязательно моделируйте угрозы для критической аутентификации, контроля доступа, бизнес-логики и ключевых потоков в приложении.
- Ограничивайте количество ресурсов на сервере, которое выделяется на одного пользователя и на одну сессию.

**Чего не стоит делать:**

- Полагаться только на обеспечение безопасности на уровне кода. Безопасный дизайн важен для создания надёжной системы в целом.
- Разрабатывать систему, не учитывая возможные атаки на неё.

## **Небезопасная конфигурация**

Небезопасная конфигурация — это ситуация, когда настройки приложения, сервера, базы данных или других компонентов системы не являются безопасными. К этой группе уязвимостей относят ненадёжные или отсутствующие настройки аутентификации, авторизации и доступа.

Предположим, что разработчик не закрыл доступ к административной панели приложения для пользователей без аутентификации или со стандартными настройками входа. Такое часто встречается у начинающих программистов. В этом случае злоумышленники легко могут взломать административную панель и изменить её настройки, подделать или украсть данные.

**Что делать:**

- Проводите безопасную настройку всех компонентов приложения и инфраструктуры, следуя рекомендациям и стандартам безопасности.
- Продумайте и поддерживайте политику настройки доступов.
- Отключайте или удаляйте ненужные функции и службы на сервере, чтобы сократить возможный спектр атак.
- Реализуйте автоматизированный процесс проверки эффективности конфигураций и настроек во всех средах.
- Регулярно проверяйте настройки на наличие уязвимостей.

**Чего не стоит делать:**

- Оставлять дефолтные пароли, настройки или ключи. Обязательно меняйте их на уникальные и сложные.
- Оставлять включёнными даже те функции и службы, что кажутся избыточными.
- Полагаться только на документацию по установке. Проверяйте и дорабатывайте настройки с учётом текущих требований безопасности.

## **Использование уязвимых или устаревших компонентов**

К этому типу уязвимостей относят случаи, когда веб-приложение использует сторонние фреймворки, библиотеки, плагины или другие компоненты, которые имеют выявленные дефекты безопасности.

У злоумышленников даже есть автоматизированные инструменты, которые помогают находить непропатченные или неправильно сконфигурированные системы. Например, поисковая система Shodan IoT ищет устройства, которые страдают от уязвимости Heartbleed, исправленной в апреле 2014 года. Удивительно, но они встречаются и в 2023 году.

**Что делать:**

- Регулярно обновляйте используемые компоненты. Следите за выпуском обновлений и исправлений, касающихся безопасности компонентов.
- Удаляйте неиспользуемые зависимости, ненужные функции, компоненты и файлы.
- Используйте источники, которые предоставляют информацию о безопасности компонентов: [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/), [Retire.js](https://retirejs.github.io/retire.js/) и другие.

**Чего не стоит делать:**

- Использовать устаревшие компоненты без обновления.
- Игнорировать предупреждения о безопасности, которые касаются используемых компонентов.

## **Ошибки идентификации и аутентификации**

Слабые пароли, недостаточная проверка подлинности, неэффективные системы учёта сеансов — всё это OWASP относит к ошибкам идентификации и аутентификации.

Предположим, что веб-приложение допускает слабые пароли, такие как «123456», «admin» или «qwerty». Злоумышленник может легко взломать аккаунт, угадав такой пароль или просто перебрав варианты.

Сюда же относятся:

- незащищённые способы восстановления паролей — например, подходы на основе знаний, когда человек должен ответить на секретный вопрос;
- отсутствие многофакторной авторизации;
- раскрытие идентификатора сессии в URL.

**Что делать:**

- Используйте сильные механизмы аутентификации, такие как двухфакторная аутентификация.
- Требуйте от пользователей создавать пароли с высокой устойчивостью к взлому, включающие в себя не только буквы, но и другие символы.
- Не раскрывайте идентификаторы сессии в URL-адресе.
- Блокируйте аккаунты после определённого количества неудачных попыток входа.

**Чего не стоит делать:**

- Разрешать пользователям использовать слабые пароли или пароли по умолчанию.
- Хранить пароли пользователей в открытом виде в базе данных. Храните хеши паролей [с солью](https://ru.wikipedia.org/wiki/%D0%A1%D0%BE%D0%BB%D1%8C_\(%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%BE%D0%B3%D1%80%D0%B0%D1%84%D0%B8%D1%8F\)).

## **Нарушения целостности программного обеспечения и данных**

К этой группе уязвимостей относят случаи, когда после обновления приложение или оборудование начинает работать неправильно. Например, роутер после обновления прошивки не требует пароля для подключения или сбрасывает его до заводского.

**Что делать:**

- Используйте проверку целостности данных, используя хеши и цифровые подписи для обнаружения несанкционированных изменений.
- Ограничивайте доступ и возможность изменения данных для неавторизованных пользователей.
- Внедряйте подробные журналы и мониторинг подозрительной активности, чтобы сохранять информацию о совершаемых пользователями действиях.

**Чего не стоит делать:**

- Хранить критически важные данные в открытом виде или без требования авторизации.
- Выдавать всем пользователям полные права на изменение данных. Всегда используйте принцип наименьших привилегий, выдавая только действительно необходимые права.
- Игнорировать предупреждения системы мониторинга о подозрительной активности. Реагируйте на них своевременно.

## **Ошибки логирования и мониторинга безопасности**

Это уязвимости, при которых система неправильно регистрирует аномальные события, касающиеся безопасности. К ним также относят отсутствие или неправильную настройку механизмов логирования и отсутствие уведомлений о подозрительных событиях.

Если в системе отсутствует мониторинг безопасности, то атаки злоумышленников могут остаться незамеченными. Это снижает вероятность быстрой реакции на возникающие инциденты, обнаружение угроз и определение их источников.

Предположим, что веб-приложение не регистрирует попытки неудачной аутентификации. Злоумышленник пытается многократно взломать аккаунт пользователя или администратора, а разработчик не замечает этого — система не фиксирует такие попытки.

**Что делать:**

- Внедрите механизмы логирования для регистрации важных событий, таких как попытки аутентификации, изменения в конфигурации и доступе к чувствительным данным.
- Установите систему мониторинга, которая анализирует логи на наличие подозрительной активности и уведомляет вас об инцидентах.
- Определите чёткие процедуры реагирования на инциденты и оповещения и обязательно расскажите о них всей команде.

**Чего не стоит делать:**

- Оставлять логирование без внимания. Регулярно анализируйте логи для выявления аномальных событий.
- Обходиться без мониторинга. Обязательно убедитесь, что система мониторинга активна и правильно настроена.
- Использовать только автоматические уведомления о состоянии системы. Регулярно вручную проверяйте состояние системы и логов.

## **Подделка запросов на стороне сервера**

Подделка запросов на стороне сервера (server-side request forgery, SSRF) — это тип уязвимости, при котором злоумышленник заставляет сервер отправлять запросы к внутренним ресурсам или внешним сайтам.

SSRF часто используется злоумышленниками для обнаружения и атаки внутренних ресурсов, к которым они обычно не имеют доступа извне. Представьте веб-приложение, которое выполняет HTTP-запросы к внешним URL-адресам на основе пользовательского ввода. Если сервер слабо защищён от SSRF, злоумышленник может ввести зловредный URL, который заставит сервер отправить запрос ко внутреннему серверу с базой данных, и получить оттуда данные.

**Что делать:**

- Ограничивайте или фильтруйте пользовательский ввод, который используется для формирования запросов.
- Используйте белый список (whitelist) разрешённых адресов, на которые сервер может отправлять запросы.
- Ограничьте и контролируйте доступ сервера к внутренним ресурсам.

**Чего не стоит делать:**

- Доверять непроверенным или неконтролируемым URL-адресам, переданным пользователем.
- Открывать доступ сервера к внутренним ресурсам без проверки.
- Использовать пользовательский ввод напрямую для формирования запросов на стороне сервера.
# 4. SQL Injection

## 🔴 Что такое SQL Injection

**SQL Injection** — это уязвимость, позволяющая злоумышленнику выполнять произвольные SQL-запросы в базе данных приложения.

## 🎯 Как работает SQL Injection

### **Базовый пример уязвимого кода**
```php
// PHP пример (для наглядности)
$username = $_POST['username'];
$password = $_POST['password'];

// УЯЗВИМЫЙ ЗАПРОС
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

**Атака:**
```
username: admin' --
password: anything
```

**Итоговый SQL:**
```sql
SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'
```
Комментарий `--` обрезает проверку пароля!

## 🚨 Типы SQL Injection

### **1. Classic UNION-based**
```sql
-- Определение количества колонок
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3-- 

-- Получение данных
' UNION SELECT username, password FROM users--
```

### **2. Error-based**
```sql
-- Извлечение данных через сообщения об ошибках
' AND 1=CAST((SELECT table_name FROM information_schema.tables) AS INT)--
```

### **3. Boolean Blind**
```sql
-- Постепенное извлечение данных по битам
' AND SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1) = 'a'--
```

### **4. Time-based Blind**
```sql
-- Использование задержек для извлечения данных
'; IF (SELECT COUNT(*) FROM users) > 0 WAITFOR DELAY '00:00:05'--
```

## 🛡️ Защита от SQL Injection

### **1. Prepared Statements (Настоящая защита)**

#### **Go + database/sql**
```go
// НЕПРАВИЛЬНО - уязвимо
func vulnerableLogin(db *sql.DB, username, password string) bool {
    query := fmt.Sprintf("SELECT * FROM users WHERE username = '%s' AND password = '%s'", username, password)
    row := db.QueryRow(query) // SQL INJECTION!
    // ...
}

// ПРАВИЛЬНО - prepared statement
func safeLogin(db *sql.DB, username, password string) (bool, error) {
    var userID int
    var dbPassword string
    
    query := "SELECT id, password FROM users WHERE username = $1"
    err := db.QueryRow(query, username).Scan(&userID, &dbPassword)
    if err != nil {
        return false, err
    }
    
    // Сравнение хешей паролей
    return checkPasswordHash(password, dbPassword), nil
}
```

#### **Python + SQLAlchemy**
```python
# НЕПРАВИЛЬНО
query = f"SELECT * FROM users WHERE username = '{username}'"
result = db.session.execute(query)

# ПРАВИЛЬНО
from sqlalchemy import text
query = text("SELECT * FROM users WHERE username = :username")
result = db.session.execute(query, {'username': username})
```

#### **Java + JDBC**
```java
// НЕПРАВИЛЬНО
String query = "SELECT * FROM users WHERE username = '" + username + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);

// ПРАВИЛЬНО
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, username);
ResultSet rs = stmt.executeQuery();
```

### **2. ORM с параметризацией**

#### **Go + GORM**
```go
type User struct {
    ID       uint
    Username string
    Password string
}

func loginWithGORM(db *gorm.DB, username, password string) (*User, error) {
    var user User
    result := db.Where("username = ?", username).First(&user)
    if result.Error != nil {
        return nil, result.Error
    }
    
    if !checkPasswordHash(password, user.Password) {
        return nil, errors.New("invalid credentials")
    }
    
    return &user, nil
}
```

#### **Django ORM**
```python
# Безопасно - ORM использует параметризацию
user = User.objects.get(username=username, password=password)

# Или с хешированием паролей
from django.contrib.auth import authenticate
user = authenticate(username=username, password=password)
```

### **3. Валидация и санитизация входных данных**

#### **Go валидация**
```go
import (
    "regexp"
    "unicode"
)

type UserInput struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

func validateUserInput(input UserInput) error {
    // Проверка длины
    if len(input.Username) < 3 || len(input.Username) > 50 {
        return errors.New("username must be between 3 and 50 characters")
    }
    
    // Проверка допустимых символов
    usernameRegex := regexp.MustCompile(`^[a-zA-Z0-9_-]+$`)
    if !usernameRegex.MatchString(input.Username) {
        return errors.New("username contains invalid characters")
    }
    
    // Проверка пароля
    if len(input.Password) < 8 {
        return errors.New("password must be at least 8 characters")
    }
    
    return nil
}

// Строгая валидация для поиска
func validateSearchTerm(term string) error {
    // Разрешаем только буквы, цифры и пробелы
    searchRegex := regexp.MustCompile(`^[a-zA-Z0-9\s]+$`)
    if !searchRegex.MatchString(term) {
        return errors.New("invalid search term")
    }
    return nil
}
```

### **4. Принцип наименьших привилегий**

#### **Создание пользователя БД с ограниченными правами**
```sql
-- Пользователь только для чтения
CREATE USER 'webapp_readonly'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT ON app_db.* TO 'webapp_readonly'@'localhost';

-- Пользователь для записи (только конкретные таблицы)
CREATE USER 'webapp_write'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE ON app_db.users TO 'webapp_write'@'localhost';
GRANT SELECT, INSERT ON app_db.logs TO 'webapp_write'@'localhost';

-- Запрещаем опасные операции
REVOKE DROP, CREATE, ALTER ON *.* FROM 'webapp_write'@'localhost';
```

## 🔧 Продвинутая защита

### **1. Web Application Firewall (WAF)**
```nginx
# nginx с ModSecurity
location / {
    ModSecurityEnabled on;
    ModSecurityConfig modsecurity.conf;
}
```

### **2. Runtime защита**
```go
type SQLInjectionDetector struct {
    patterns []*regexp.Regexp
}

func NewSQLInjectionDetector() *SQLInjectionDetector {
    return &SQLInjectionDetector{
        patterns: []*regexp.Regexp{
            regexp.MustCompile(`(?i)(union\s+select)`),
            regexp.MustCompile(`(?i)(drop\s+table)`),
            regexp.MustCompile(`(?i)(insert\s+into)`),
            regexp.MustCompile(`(?i)(sleep\s*\(\d+\))`),
            regexp.MustCompile(`(['";]+\s*OR\s*['";]+)`),
        },
    }
}

func (d *SQLInjectionDetector) Detect(input string) bool {
    for _, pattern := range d.patterns {
        if pattern.MatchString(input) {
            return true
        }
    }
    return false
}

// Middleware для проверки
func SQLInjectionMiddleware(next http.Handler) http.Handler {
    detector := NewSQLInjectionDetector()
    
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Проверяем query parameters
        for _, values := range r.URL.Query() {
            for _, value := range values {
                if detector.Detect(value) {
                    http.Error(w, "Potential SQL injection detected", http.StatusBadRequest)
                    return
                }
            }
        }
        
        // Проверяем POST parameters
        if err := r.ParseForm(); err == nil {
            for _, values := range r.PostForm {
                for _, value := range values {
                    if detector.Detect(value) {
                        http.Error(w, "Potential SQL injection detected", http.StatusBadRequest)
                        return
                    }
                }
            }
        }
        
        next.ServeHTTP(w, r)
    })
}
```

## 🎯 Реальные примеры атак

### **Атака на интернет-магазин**
```sql
-- Исходный запрос
SELECT * FROM products WHERE category = 'electronics'

-- Атакованный запрос
SELECT * FROM products WHERE category = 'electronics' 
UNION SELECT username, password, NULL, NULL FROM users--'
```

### **Кража сессий**
```sql
-- Уязвимый код обновления сессии
UPDATE sessions SET user_id = 1234 WHERE session_id = 'input_session_id'

-- Атака
input_session_id: ' OR 1=1; UPDATE sessions SET user_id = 1 WHERE session_id = 'current_session
```

## 📊 Мониторинг и обнаружение

### **Логирование подозрительных запросов**
```go
type QueryLogger struct {
    logger *log.Logger
}

func (ql *QueryLogger) LogQuery(query string, args []interface{}, duration time.Duration) {
    // Логируем все запросы
    ql.logger.Printf("QUERY: %s | ARGS: %v | DURATION: %v", query, args, duration)
    
    // Обнаружение потенциальных инъекций
    if ql.isSuspicious(query, args) {
        ql.logger.Printf("SUSPICIOUS QUERY DETECTED: %s", query)
        // Отправка алерта
        ql.sendAlert(query, args)
    }
}

func (ql *QueryLogger) isSuspicious(query string, args []interface{}) bool {
    suspiciousPatterns := []string{
        " UNION ", " OR 1=1", " AND 1=1", 
        " WAITFOR DELAY ", " BENCHMARK ", 
        " INFORMATION_SCHEMA", "xp_cmdshell",
    }
    
    queryLower := strings.ToLower(query)
    for _, pattern := range suspiciousPatterns {
        if strings.Contains(queryLower, strings.ToLower(pattern)) {
            return true
        }
    }
    
    return false
}
```

## 🚀 Best Practices

### **1. Всегда используйте prepared statements**
```go
// ХОРОШО
func getUserByEmail(db *sql.DB, email string) (*User, error) {
    query := "SELECT id, name, email FROM users WHERE email = ?"
    row := db.QueryRow(query, email)
    // ...
}

// ПЛОХО
func vulnerableGetUser(db *sql.DB, email string) (*User, error) {
    query := fmt.Sprintf("SELECT id, name, email FROM users WHERE email = '%s'", email)
    row := db.QueryRow(query) // SQL INJECTION!
    // ...
}
```

### **2. Используйте строгую типизацию**
```go
func getUserByID(db *sql.DB, userID int) (*User, error) {
    var user User
    err := db.QueryRow(
        "SELECT id, name, email FROM users WHERE id = $1", 
        userID, // Тип проверен на этапе компиляции
    ).Scan(&user.ID, &user.Name, &user.Email)
    
    return &user, err
}
```

### **3. Минимизируйте привилегии БД**
```sql
-- Создаём пользователя только с необходимыми правами
CREATE USER 'api_user'@'%' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT ON myapp.users TO 'api_user'@'%';
GRANT SELECT ON myapp.products TO 'api_user'@'%';
REVOKE DELETE, DROP, CREATE, ALTER ON *.* FROM 'api_user'@'%';
```

### **4. Регулярно обновляйте зависимости**
```bash
# Проверка уязвимостей в Go
go list -json -m all | nancy scan

# В Node.js
npm audit

# В Python
safety check
```

## 💡 Заключение

**SQL Injection полностью предотвращается** с помощью:
- ✅ **Prepared Statements** для всех запросов
- ✅ **ORM** с параметризацией
- ✅ **Валидации** входных данных
- ✅ **Принципа наименьших привилегий**
- ✅ **Регулярного аудита** безопасности

Никогда не доверяйте пользовательскому вводу и всегда используйте параметризованные запросы!
# 5. loot box / tranactional loot box
# 6. join sql

## 🎯 Что такое JOIN

**JOIN** — операция SQL, которая объединяет строки из двух или более таблиц на основе связанного столбца между ними.

## 📊 Типы JOIN

### **Базовые типы JOIN**
```sql
-- INNER JOIN - только совпадающие строки
SELECT * FROM table1 
INNER JOIN table2 ON table1.id = table2.table1_id;

-- LEFT JOIN - все строки из левой таблицы + совпадающие из правой
SELECT * FROM table1 
LEFT JOIN table2 ON table1.id = table2.table1_id;

-- RIGHT JOIN - все строки из правой таблицы + совпадающие из левой  
SELECT * FROM table1 
RIGHT JOIN table2 ON table1.id = table2.table1_id;

-- FULL OUTER JOIN - все строки из обеих таблиц
SELECT * FROM table1 
FULL OUTER JOIN table2 ON table1.id = table2.table1_id;

-- CROSS JOIN - декартово произведение (все комбинации)
SELECT * FROM table1 
CROSS JOIN table2;
```

## 🗄️ Пример базы данных

```sql
-- Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Таблица заказов
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount DECIMAL(10,2),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Таблица товаров
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2)
);

-- Таблица связи заказов и товаров
CREATE TABLE order_items (
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER DEFAULT 1,
    PRIMARY KEY (order_id, product_id)
);
```

## 🔍 INNER JOIN

### **Базовый INNER JOIN**
```sql
-- Пользователи с их заказами
SELECT 
    u.name,
    u.email,
    o.amount,
    o.status
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

### **INNER JOIN с несколькими таблицами**
```sql
-- Детали заказов с информацией о пользователях и товарах
SELECT 
    u.name AS customer_name,
    o.id AS order_id,
    p.name AS product_name,
    oi.quantity,
    (p.price * oi.quantity) AS total_price
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

### **INNER JOIN с фильтрацией**
```sql
-- Только завершенные заказы
SELECT 
    u.name,
    o.amount,
    o.created_at
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
ORDER BY o.amount DESC;
```

## 🔍 LEFT JOIN

### **Базовый LEFT JOIN**
```sql
-- Все пользователи и их заказы (если есть)
SELECT 
    u.name,
    u.email,
    COALESCE(o.amount, 0) AS order_amount,
    COALESCE(o.status, 'no orders') AS order_status
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### **LEFT JOIN для поиска отсутствующих данных**
```sql
-- Пользователи без заказов
SELECT 
    u.id,
    u.name,
    u.email
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### **LEFT JOIN с агрегацией**
```sql
-- Количество заказов у каждого пользователя
SELECT 
    u.name,
    COUNT(o.id) AS order_count,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
ORDER BY total_spent DESC;
```

## 🔍 MULTIPLE JOIN

### **Сложные связи между таблицами**
```sql
-- Полная информация о заказах
SELECT 
    u.name AS customer,
    o.id AS order_id,
    o.created_at AS order_date,
    p.name AS product,
    oi.quantity,
    c.name AS category
FROM orders o
INNER JOIN users u ON o.user_id = u.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
LEFT JOIN categories c ON p.category_id = c.id
WHERE o.status = 'completed'
ORDER BY o.created_at DESC;
```

## 🔍 SELF JOIN

### **Иерархические данные**
```sql
-- Таблица сотрудников с менеджерами
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    manager_id INTEGER REFERENCES employees(id)
);

-- Сотрудники и их менеджеры
SELECT 
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### **Поиск пар**
```sql
-- Найти пользователей с одинаковыми именами
SELECT 
    u1.name,
    u1.email AS email1,
    u2.email AS email2
FROM users u1
INNER JOIN users u2 ON u1.name = u2.name AND u1.id < u2.id;
```

## 🔍 CROSS JOIN

### **Декартово произведение**
```sql
-- Все комбинации размеров и цветов
SELECT 
    s.size_name,
    c.color_name
FROM sizes s
CROSS JOIN colors c;
```

### **Генерация данных**
```sql
-- Генерация дат на неделю вперед
WITH dates AS (
    SELECT CURRENT_DATE + i AS date
    FROM generate_series(0, 6) AS i
)
SELECT 
    d.date,
    u.name
FROM dates d
CROSS JOIN users u
ORDER BY d.date, u.name;
```

## 🔍 JOIN с агрегатными функциями

### **Статистика по пользователям**
```sql
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS total_orders,
    SUM(o.amount) AS total_amount,
    AVG(o.amount) AS avg_order_amount,
    MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 0  -- Только пользователи с заказами
ORDER BY total_amount DESC;
```

### **Топ товаров по продажам**
```sql
SELECT 
    p.name AS product_name,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * p.price) AS total_revenue,
    COUNT(DISTINCT o.id) AS order_count
FROM products p
INNER JOIN order_items oi ON p.id = oi.product_id
INNER JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed'
GROUP BY p.id, p.name
ORDER BY total_revenue DESC
LIMIT 10;
```

## 🔍 JOIN с подзапросами

### **JOIN с производными таблицами**
```sql
-- Пользователи с их последним заказом
SELECT 
    u.name,
    u.email,
    latest_order.amount,
    latest_order.created_at
FROM users u
INNER JOIN (
    SELECT 
        user_id,
        amount,
        created_at,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
    FROM orders
) latest_order ON u.id = latest_order.user_id AND latest_order.rn = 1;
```

### **JOIN с CTE (Common Table Expressions)**
```sql
WITH user_stats AS (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
),
top_users AS (
    SELECT 
        user_id,
        total_spent
    FROM user_stats
    WHERE total_spent > 1000
)
SELECT 
    u.name,
    us.order_count,
    us.total_spent
FROM users u
INNER JOIN user_stats us ON u.id = us.user_id
INNER JOIN top_users tu ON u.id = tu.user_id
ORDER BY us.total_spent DESC;
```

## ⚡ Оптимизация JOIN

### **Индексы для JOIN**
```sql
-- Обязательные индексы для эффективных JOIN
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_products_category_id ON products(category_id);
```

### **EXPLAIN для анализа JOIN**
```sql
-- Анализ производительности запроса
EXPLAIN ANALYZE
SELECT 
    u.name,
    o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01';
```

## 🚨 Распространенные ошибки

### **1. Cartesian Product (случайный CROSS JOIN)**
```sql
-- НЕПРАВИЛЬНО - забыли условие JOIN
SELECT * FROM users, orders;  -- Декартово произведение!

-- ПРАВИЛЬНО
SELECT * FROM users 
INNER JOIN orders ON users.id = orders.user_id;
```

### **2. Неоднозначные имена столбцов**
```sql
-- НЕПРАВИЛЬНО - какой id?
SELECT id, name FROM users 
INNER JOIN orders ON users.id = orders.user_id;

-- ПРАВИЛЬНО - указываем таблицу
SELECT users.id, users.name, orders.amount 
FROM users 
INNER JOIN orders ON users.id = orders.user_id;
```

### **3. Неправильное условие JOIN**
```sql
-- НЕПРАВИЛЬНО - неправильная логика
SELECT * FROM users 
INNER JOIN orders ON users.id = orders.amount;  -- Бессмыслица

-- ПРАВИЛЬНО - связываем по внешнему ключу
SELECT * FROM users 
INNER JOIN orders ON users.id = orders.user_id;
```

## 🎯 Best Practices

### **1. Используйте псевдонимы таблиц**
```sql
-- ХОРОШО
SELECT 
    u.name,
    o.amount,
    p.name AS product_name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;

-- ПЛОХО
SELECT 
    users.name,
    orders.amount,
    products.name
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN order_items ON orders.id = order_items.order_id
INNER JOIN products ON order_items.product_id = products.id;
```

### **2. Выбирайте только нужные столбцы**
```sql
-- ХОРОШО - только нужные данные
SELECT 
    u.name,
    o.amount,
    o.status
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ПЛОХО - все столбцы
SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

### **3. Правильно выбирайте тип JOIN**
```sql
-- Нужны все пользователи, даже без заказов?
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id;  -- Да, LEFT JOIN

-- Только пользователи с заказами?
SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;  -- Да, INNER JOIN
```

## 💡 Продвинутые техники

### **LATERAL JOIN (PostgreSQL)**
```sql
-- Для каждого пользователя получить 3 последних заказа
SELECT 
    u.name,
    recent_orders.amount,
    recent_orders.created_at
FROM users u
CROSS JOIN LATERAL (
    SELECT amount, created_at
    FROM orders 
    WHERE user_id = u.id 
    ORDER BY created_at DESC 
    LIMIT 3
) recent_orders;
```

### **NATURAL JOIN (осторожно!)**
```sql
-- Автоматически JOIN по одинаковым именам столбцов
SELECT * FROM users 
NATURAL JOIN orders;

-- Эквивалентно:
SELECT * FROM users 
INNER JOIN orders USING (id);  -- Опасно если совпадение неверное!
```

**JOIN** — мощный инструмент для объединения данных из разных таблиц. Правильное использование JOIN позволяет создавать сложные запросы и получать именно те данные, которые нужны.
# 7. left/right/inner join sql
# 8. sql index
# 9. виды индексов
# 10. авто индексы
# 11. sql explain
# 12. партиционыирование sql


[  
00:27](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=27s) – Начало работы Уверенность: Средняя. Правильность: Подключил необходимые пакеты, но были небольшие задержки. [01:43](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=103s) – Обработка ошибок Уверенность: Средняя. Правильность: Верно обработал ошибки, но не сразу учёл все нюансы. [05:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=319s) – Закрытие тела запроса Уверенность: Средняя. Правильность: Пропустил необходимость закрытия тела запроса, но потом исправил. [06:28](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=388s) – Использование defer Уверенность: Средняя. Правильность: Правильно применил defer, но не сразу учёл все кейсы. [08:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=499s) – Модификация программы Уверенность: Средняя. Правильность: Модифицировал программу для работы с горутинами, но были ошибки. [13:03](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=783s) – Работа с каналами Уверенность: Средняя. Правильность: Выбрал небуферизированные каналы, объяснил выбор. [15:09](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=909s) – Обработка неизвестного количества данных Уверенность: Средняя. Правильность: Предложил решение для обработки данных из внешнего источника. [18:19](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1099s) – Параллельное выполнение Уверенность: Средняя. Правильность: Реализовал воркер-пул, но были недочёты в обработке контекста. [26:37](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1597s) – Код-ревью Уверенность: Высокая Правильность: Нашёл все основные ошибки [31:45](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1905s) – Хранение данных Уверенность: Высокая Правильность: Обсудил хранение паролей, предложил использование секретов. [33:03](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=1983s) – Обсуждение структуры данных Уверенность: Высокая Правильность: Предложил вынести структуру в отдельный файл. [34:40](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2080s) – Поиск оптимального решения Уверенность: Средняя. Правильность: Обсудил архитектурные подходы, но не сразу предложил паттерн Outbox [36:24](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2184s) – Реализация стратегии Уверенность: Средняя. Правильность: Планировал использование стратегии, но не реализовал. [37:01](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2221s) – Анализ кода и обработка ошибок Уверенность: Высокая Правильность: Обнаружил некорректное использование токена. [39:42](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2382s) – Проверка подключения к базе Уверенность: Средняя. Правильность: Обсудил критичность подключения, но не предложил решение. [40:49](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2449s) – Избыточная обработка ошибок Уверенность: Средняя. Правильность: Обнаружил избыточность, но не исправил. [42:05](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2525s) – Проблемы с построением запросов Уверенность: Средняя. Правильность: Указал на риски инъекций, но не предложил решение. [43:21](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2601s) – Логика работы с соединениями Уверенность: Средняя. Правильность: Отметил неэффективность, предложил пул соединений. [45:47](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2747s) – Архитектура системы Уверенность: Средняя. Правильность: Предложил Kafka, но не учёл ограничения. [48:07](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=2887s) – Синхронное взаимодействие с аналитикой Уверенность: Средняя. Правильность: Обсудил синхронную обработку, но не предложил решение. [52:33](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3153s) – Последовательность действий Уверенность: Средняя. Правильность: Описал последовательность, но не учёл все нюансы. [55:30](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3330s) – Управление нагрузкой Уверенность: Выше среднего. Правильность: Предложил увеличение воркеров, но не детализировал. [57:49](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=3469s) – Создание таблиц Уверенность: Средняя. Правильность: Создал таблицы, но были ошибки в полях. [01:09:38](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4178s) – Тестирование запросов Уверенность: Средняя. Правильность: Протестировал запросы, но не все изменения. [01:10:45](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4245s) – Введение в индексы Уверенность: Высокая Правильность: Корректно описал индексы. [01:11:34](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4294s) – Применение индексов Уверенность: Средняя. Правильность: Правильно объяснил использование индексов. [01:12:36](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4356s) – Автоматические индексы Уверенность: Высокая. Правильность: Упомянул B-Tree индексы. [01:13:07](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4387s) – Анализ запросов Уверенность: Выше среднего. Правильность: Описал инструмент EXPLAIN, но не все метрики. [01:15:09](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4509s) – Оптимизация запросов Уверенность: Выше среднего. Правильность: Предложил партиционирование, но не детализировал. [01:17:41](https://www.youtube.com/watch?v=-8JlOr3Z0eA&t=4661s) – Репликация Уверенность: Средняя. Правильность: Обсудил репликацию, но не учёл частую запись.

##