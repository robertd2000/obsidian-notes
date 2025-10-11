
# 1 Инициализация мапы
# 2 make map сколько аргументов
- **`make(map[T]U)`** - 1 аргумент, авто-ёмкость
    
- **`make(map[T]U, capacity)`** - 2 аргумента, указание ёмкости
    
- **Указывайте ёмкость** когда известно приблизительное количество элементов
    
- **Не указывайте** для маленьких или непредсказуемых map
# 3 пустой интерфейс

## Что это такое

Пустой интерфейс `interface{}` — это интерфейс без методов. Он может хранить значения любого типа, поскольку все типы неявно удовлетворяют интерфейсу без методов.

## Основные свойства

- **Универсальный контейнер** — принимает значения любого типа
- **Тип во время выполнения** — информация о типе сохраняется и доступна через механизм reflection
- **Нулевое значение** — `nil`

## Когда используется

### 1. Работа с динамическими данными
- JSON парсинг без заранее известной структуры
- Данные из внешних источников с неизвестной схемой

### 2. Универсальные функции
Функции, которые должны работать с разными типами данных:
```go
func PrintAnything(v interface{}) {
    fmt.Println(v)
}
```

### 3. Коллекции разных типов
Слайсы, мапы или каналы, содержащие различные типы данных:
```go
var items []interface{}
items = append(items, 42, "text", true)
```

## Как работать с пустым интерфейсом

### Type Assertion (утверждение типа)
Проверка и преобразование к конкретному типу:
```go
value, ok := someVar.(string)
if ok {
    // работаем со string
}
```

### Type Switch
Обработка разных типов в switch:
```go
switch v := value.(type) {
case int:
    // обработка int
case string:
    // обработка string
default:
    // другие типы
}
```

## Особенности и предостережения

- **Потеря безопасности типов** — компилятор не может проверить корректность типов
- **Необходимость проверок** — всегда нужно проверять реальный тип перед использованием
- **Производительность** — дополнительные проверки типов во время выполнения

## Современная альтернатива

В Go 1.18+ вместо пустого интерфейса часто лучше использовать дженерики (generics), которые обеспечивают типобезопасность на этапе компиляции.

## Главное правило

Используйте пустой интерфейс только когда действительно необходима работа с неизвестными типами. В остальных случаях предпочитайте конкретные типы или дженерики для безопасности и производительности.
# 4 размер пустого интерфейса

## Базовая структура

Пустой интерфейс `interface{}` состоит из двух указателей:

- **Указатель на тип** (type descriptor) - 8 байт
- **Указатель на данные** (value pointer) - 8 байт

**Итого: 16 байт** на 64-битной архитектуре

## Техническая реализация

```go
type iface struct {
    tab  *itab          // 8 байт - информация о типе
    data unsafe.Pointer // 8 байт - указатель на данные
}
```

## Примеры размеров

```go
var a interface{}        // 16 байт (даже когда nil)
var b interface{} = 42   // 16 байт + память под int (8 байт)
var c interface{} = "hello" // 16 байт + память под строку
```

## Ключевые моменты

1. **Фиксированный размер** - всегда 16 байт для самого интерфейса
2. **Дополнительная память** - данные хранятся отдельно
3. **Small values копируются** - маленькие значения (≤ 8 байт) могут храниться непосредственно в data pointer через оптимизации
4. **Large values аллоцируются** - большие значения выделяются в куче

## Сравнение с конкретными типами

- `int` - 8 байт
- `string` - 16 байт (уже как интерфейс!)
- `interface{}` - 16 байт + данные

## Вывод

Пустой интерфейс добавляет **16 байт накладных расходов** плюс стоимость хранения самих данных. Это важно учитывать в высоконагруженных приложениях.
# 5 пустая структура

## Суть пустой структуры

`struct{}` — это специальный тип в Go, который **не занимает памяти**, но при этом **соответствует всем правилам системы типов**.

## Ключевые свойства

### **1. Нулевой размер**
```go
size := unsafe.Sizeof(struct{}{})  // 0 байт
```
- Не требует выделения памяти
- Массивы/слайсы пустых структур тоже имеют размер 0

### **2. Уникальные адреса**
```go
a, b := struct{}{}, struct{}{}
&a != &b  // true - разные адреса
```
**Важно**: Это логические адреса в системе типов, а не указатели на реальную память.

### **3. Все экземпляры идентичны**
```go
struct{}{} == struct{}{}  // true
```

## Как это технически работает

### **Компиляторная магия**
- Компилятор создаёт "виртуальные адреса" для каждой переменной `struct{}`
- Эти адреса не соответствуют реальным ячейкам памяти
- Система типов считает, что у переменных есть адреса, но runtime не выделяет под них память

### **Глобальный синглтон**
Как указано в статье Дейва Чейни, все пустые структуры в конечном счёте ссылаются на специальный нулевой байт в памяти, но каждая переменная получает свой уникальный "логический адрес" в контексте системы типов.

## Основные применения

### **1. Каналы-сигналы**
```go
done := make(chan struct{})
go func() {
    // работа
    done <- struct{}{}  // pure signal - нет данных
}()
```

### **2. Множества (Set)**
```go
set := make(map[string]struct{})
set["key"] = struct{}{}  // экономнее чем map[string]bool
```

### **3. Методы без состояния**
```go
type Processor struct{}  // не занимает память у экземпляров

func (p Processor) Process() { /* ... */ }
```

## Преимущества

### **Экономия памяти**
- **Map с миллионом элементов**: экономия ~1MB по сравнению с `bool`
- **Массивы/слайсы**: нулевое потребление памяти
- **Каналы**: только управляющие структуры, без payload

### **Семантическая ясность**
```go
// Понятно, что это сигнал, а не передача данных
signal <- struct{}{}

// В отличие от менее ясного:
signal <- true
```

## Особенности реализации

### **Отличие от других языков**
- **C++**: empty classes имеют размер ≥ 1 байт
- **Go**: смог обойти это ограничение благодаря управлению памятью на уровне runtime

### **Поведение в структурах**
```go
type Example struct {
    A struct{}  // 0 байт
    B int64     // 8 байт
    C struct{}  // 0 байт
}
// Размер структуры = 8 байт (только под B)
```

## Практические выводы

1. **Используйте для сигналов** — когда важен факт события, а не данные
2. **Используйте для множеств** — экономия памяти в больших map
3. **Используйте для stateless методов** — когда нужны методы, но не нужно состояние
4. **Не бойтесь "противоречия"** — уникальные адреса при нулевом размере это осознанный компромисс дизайна Go

## Философский аспект

Пустая структура в Go — это элегантное решение, которое:
- Сохраняет **безопасность типов** (уникальные адреса)
- Обеспечивает **эффективность** (нулевое потребление памяти)  
- Даёт **семантическую ясность** (чёткое обозначение намерений)

Это не баг, а фича, которая демонстрирует тщательно продуманный дизайн системы типов Go.
# 6 если сделать тип значений у мапы интерфейс и передавать туда пустую структуру - будет она занимать память?

## Короткий ответ: **ДА, будет занимать память**

## Что происходит в памяти

```go
m := make(map[string]interface{})
m["key"] = struct{}{}
```

### **Размеры:**
- `struct{}{}` сам по себе: **0 байт**
- `interface{}` обёртка: **16 байт**
- Map entry целиком: **~48+ байт**

## Технические детали

### **1. Interface{} добавляет накладные расходы**
```go
type iface struct {
    tab  *itab          // 8 байт - информация о типе
    data unsafe.Pointer // 8 байт - указатель на данные
}
```
**Итого: 16 байт на каждое значение в map**

### **2. Пустая структура в interface{}**
```go
// Компилятор создаёт:
var emptyStruct struct{}
iface := interface{}{
    tab:  &struct{}_type_info,  // 8 байт
    data: &emptyStruct,          // 8 байт (указатель на глобальную пустую структуру)
}
```

### **3. Map entry структура**
```go
type hmap struct {
    // ... поля map
}

// Каждая запись в map:
type bmap struct {
    keys   [8]keyType    // ~8 байт на ключ
    values [8]valueType  // 16 байт на значение (interface{})
    // + overhead хэш-таблицы
}
```

## Сравнение подходов

### **Вариант 1: map[string]struct{}**
```go
m1 := make(map[string]struct{})
m1["key"] = struct{}{}
// Память: ~48 байт на entry (только overhead map + ключ)
```

### **Вариант 2: map[string]interface{} с struct{}**
```go
m2 := make(map[string]interface{})
m2["key"] = struct{}{}
// Память: ~48 + 16 = ~64 байт на entry
```

## Практический тест
```go
func main() {
    m1 := make(map[string]struct{})
    m2 := make(map[string]interface{})
    
    for i := 0; i < 1000; i++ {
        key := fmt.Sprintf("key%d", i)
        m1[key] = struct{}{}
        m2[key] = struct{}{}
    }
    
    // m2 займёт значительно больше памяти из-за interface{} обёртки
}
```

## Вывод

**Используйте `map[T]struct{}` вместо `map[T]interface{}`** когда нужно множество (set), чтобы избежать накладных расходов interface{}.
# 7 сложность работы с мапой

# 8 как fmt.Println(n) выведет мапу n - в отсортированном по ключам порядке

# 9 каналы

# 10 как канал устроен под капотом

# 11 чем обеспечивается потокобезопасность канала

## Основной механизм: **Мьютекс в структуре hchan**

```go
type hchan struct {
    lock mutex  // этот мьютекс защищает все поля структуры
    // ... остальные поля
}
```

## Как работает мьютекс канала

### **1. Единый мьютекс на весь канал**
- Все операции (send/receive/close) захватывают один мьютекс
- Только одна горутина может работать с каналом в момент времени
- Гарантируется атомарность операций

### **2. Пример защиты операций**
```go
// Операция отправки
func chansend(c *hchan, ep unsafe.Pointer, block bool) bool {
    lock(&c.lock)  // ЗАХВАТ МЬЮТЕКСА
    
    // ... логика отправки
    
    unlock(&c.lock)  // ОСВОБОЖДЕНИЕ МЬЮТЕКСА
    return true
}

// Операция получения
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) bool {
    lock(&c.lock)  // ЗАХВАТ ТОГО ЖЕ МЬЮТЕКСА
    
    // ... логика получения
    
    unlock(&c.lock)
    return true
}
```

## Сценарии потокобезопасности

### **1. Конкурирующие отправки**
```go
ch := make(chan int, 1)

go func() { ch <- 1 }()  // Горутина 1
go func() { ch <- 2 }()  // Горутина 2

// Мьютекс гарантирует, что только одна горутина
// сможет отправить данные в конкретный момент
```

### **2. Конкурирующие получения**
```go
go func() { <-ch }()  // Горутина 1
go func() { <-ch }()  // Горутина 2  
go func() { <-ch }()  // Горутина 3

// Мьютекс гарантирует порядок доступа к данным
```

### **3. Смешанные операции**
```go
go func() { ch <- 1 }()    // Отправитель
go func() { value := <-ch }()  // Получатель
go func() { close(ch) }()  // Закрытие

// Все операции защищены одним мьютексом
```

## Особые случаи

### **Быстрый путь (fast path) для буферизованных каналов**
```go
// Когда буфер не полон и не пуст
if c.qcount < c.dataqsiz {
    // Быстрая вставка в буфер без полной блокировки
    // Но всё равно с атомарными операциями
}
```

### **Прямая передача данных**
```go
// Когда есть ждущий отправитель/получатель
if sg := c.recvq.dequeue(); sg != nil {
    // Прямая передача данных от отправителя к получателю
    // Всё ещё под защитой мьютекса
}
```

## Что именно защищает мьютекс

### **Защищаемые поля:**
- `buf` - указатель на буфер
- `qcount`, `sendx`, `recvx` - счётчики и индексы
- `recvq`, `sendq` - очереди ожидания
- `closed` - флаг закрытия

### **Пример конфликта без мьютекса:**
```go
// БЕЗ МЬЮТЕКСА (гипотетически):
// Горутина 1: проверяет qcount < dataqsiz
// Горутина 2: одновременно уменьшает qcount
// РЕЗУЛЬТАТ: состояние гонки - обе могут попытаться писать в один слот
```

## Взаимодействие с планировщиком

### **Блокирующие операции**
```go
func chansend(c *hchan, ep unsafe.Pointer, block bool) bool {
    lock(&c.lock)
    
    if !block && c.closed == 0 && ((c.dataqsiz == 0 && c.recvq.first == nil) ||
        (c.dataqsiz > 0 && c.qcount == c.dataqsiz)) {
        unlock(&c.lock)
        return false
    }
    
    // Если нельзя отправить - добавляем в очередь ожидания
    gp := getg()
    mysg := acquireSudog()
    mysg.g = gp
    c.sendq.enqueue(mysg)
    
    // БЛОКИРОВКА ГОРУТИНЫ (отпускаем мьютекс перед блокировкой)
    unlock(&c.lock)
    gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
    
    // Пробуждение - снова захватываем мьютекс
    lock(&c.lock)
    // ... продолжение логики
}
```

## Гарантии потокобезопасности

### **1. Атомарность операций**
- Каждая операция send/receive/close атомарна
- Не может быть частично выполненной операции

### **2. Последовательность событий**
- Порядок отправки/получения сохраняется
- Закрытие канала видно всем горутинам одновременно

### **3. Корректность данных**
- Нет гонок данных (data races) при доступе к буферу
- Очереди ожидания защищены от конкурентного доступа

## Пример демонстрирующий потокобезопасность

```go
func demonstrateChannelSafety() {
    ch := make(chan int)
    var results []int
    
    // Множество писателей
    for i := 0; i < 10; i++ {
        go func(val int) {
            ch <- val  // Все горутины безопасно пишут в один канал
        }(i)
    }
    
    // Читатель
    for i := 0; i < 10; i++ {
        val := <-ch
        results = append(results, val)
    }
    
    // Все данные будут получены без потерь
    fmt.Println("Received:", len(results), "values") // Всегда 10
}
```

## Итог

**Потокобезопасность каналов обеспечивается:**
- **Единым мьютексом** на структуру `hchan`
- **Строгой последовательностью** захвата/освобождения мьютекса
- **Атомарными операциями** над состоянием канала
- **Интеграцией с планировщиком** для блокирующих операций

Это делает каналы идеальным примитивом для безопасной коммуникации между горутинами без дополнительных синхронизаций.
# 12 мьютексы

## Базовая структура

```go
type Mutex struct {
    state int32  // состояние мьютекса (32 бита)
    sema  uint32 // семафор для блокировок
}
```

## Состояние state (32 бита)

```
Bit layout:
| 0 | 1-30 | 31 |
|---|------|----|
| W |  W   | L  |

L (1 бит) - заблокирован ли мьютекс (0/1)
W (31 бит) - количество ждущих горутин
```

## Основные режимы работы

### **1. Normal mode**
- Конкуренция за мьютекс
- Ждущие горутины становятся в очередь FIFO
- Пробуждение в порядке очереди

### **2. Starvation mode** 
- Если горутина ждёт мьютекс > 1ms
- Мьютекс переходит в starvation mode
- Мьютекс передаётся следующей ждущей горутине напрямую

## Алгоритм работы

### **Lock()**
```go
func (m *Mutex) Lock() {
    // Fast path: сразу захватываем если мьютекс свободен
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
        return
    }
    
    // Slow path: конкуренция или уже заблокирован
    m.lockSlow()
}
```

### **Unlock()**
```go
func (m *Mutex) Unlock() {
    // Fast path: просто снимаем блокировку
    new := atomic.AddInt32(&m.state, -mutexLocked)
    
    // Slow path: есть ждущие горутины
    if new != 0 {
        m.unlockSlow(new)
    }
}
```

## Ключевые оптимизации

### **Spin waiting (активное ожидание)**
- Горутина крутится в цикле ~4 итерации
- Пытается захватить мьютекс без перехода в sleep
- Эффективно при коротких блокировках

### **Atomic операции**
- Использует `atomic.CompareAndSwapInt32`
- Lock-free при отсутствии конкуренции
- Минимальные overhead в fast path

## Семафорная система

- `sema` — ссылка на семафор планировщика
- Ждущие горутины блокируются через `runtime_SemacquireMutex`
- Пробуждение через `runtime_Semrelease`

## Преимущества реализации

- **Fast path**: 1 атомарная операция при отсутствии конкуренции
- **Адаптивный**: сам переключается между normal/starvation mode
- **Справедливый**: предотвращает голодание горутин
- **Эффективный**: spin + sleep стратегия

**Итог**: Умная гибридная реализация, сочетающая скорость atomic операций и справедливость очередей ожидания.
# 13 атомики

# 14 паттерны конкурентного программирования

# 15 pipeline

# 16 worker pull

# 17 rate limiter

# 18 semaphore

# 19 Операции над закрытыми каналами

# 20 Операции над nil каналами

# 21 deadlock

## Что это

**Deadlock** — ситуация, когда две или более горутин бесконечно ждут друг друга, потому что каждая держит ресурс, нужный другой.

## Условия возникновения

Для deadlock необходимы **все 4 условия**:

1. **Взаимное исключение** (Mutual Exclusion) — ресурс может использоваться только одной горутиной
2. **Удержание и ожидание** (Hold and Wait) — горутина держит один ресурс и ждёт другой
3. **Отсутствие вытеснения** (No Preemption) — ресурс нельзя отнять у горутины
4. **Круговое ожидание** (Circular Wait) — горутины ждут друг друга по кругу

## Классический пример

```go
var mu1, mu2 sync.Mutex

func goroutine1() {
    mu1.Lock()    // Захватываем первый мьютекс
    time.Sleep(1 * time.Millisecond)
    
    mu2.Lock()    // Ждём второй мьютекс (но его держит goroutine2)
    fmt.Println("goroutine1")
    
    mu2.Unlock()
    mu1.Unlock()
}

func goroutine2() {
    mu2.Lock()    // Захватываем второй мьютекс  
    time.Sleep(1 * time.Millisecond)
    
    mu1.Lock()    // Ждём первый мьютекс (но его держит goroutine1)
    fmt.Println("goroutine2")
    
    mu1.Unlock()
    mu2.Unlock()
}

// Запуск:
go goroutine1()
go goroutine2()
// Обе горутины заблокированы навсегда
```

## Другие сценарии

### **Каналы**
```go
ch1 := make(chan int)
ch2 := make(chan int)

go func() {
    <-ch1        // Ждём данные из ch1
    ch2 <- 42    // Отправляем в ch2
}()

go func() {  
    <-ch2        // Ждём данные из ch2  
    ch1 <- 100   // Отправляем в ch1
}()

// Обе горутины ждут друг друга
```

### **WaitGroup неправильное использование**
```go
var wg sync.WaitGroup

func process() {
    wg.Add(1)
    // Забыли wg.Done() - вечная блокировка
}

wg.Wait() // Deadlock
```

## Обнаружение в Go

### **1. Детектор deadlock в runtime**
Go имеет встроенный детектор, который может обнаружить некоторые deadlock:
```go
go func() {
    mu.Lock()
    // Забыли Unlock() - при сборке мусора может обнаружиться deadlock
}()
```

### **2. Тестовый режим**
```bash
go test -deadlock
```

## Профилактика

### **1. Единый порядок захвата мьютексов**
```go
// ПРАВИЛЬНО - всегда захватываем в одном порядке
func safeOperation() {
    mu1.Lock()
    mu2.Lock()
    // работа
    mu2.Unlock()
    mu1.Unlock()
}
```

### **2. Использование select с timeout**
```go
select {
case <-ch:
    // успех
case <-time.After(time.Second):
    // таймаут - избежали deadlock
}
```

### **3. TryLock когда возможно**
```go
if mu.TryLock() {
    defer mu.Unlock()
    // работа
} else {
    // альтернативная логика
}
```

### **4. Context для отмены**
```go
func operation(ctx context.Context) error {
    select {
    case <-ch:
        return nil
    case <-ctx.Done():
        return ctx.Err() // выходим по отмене
    }
}
```

## Отладка

### **1. pprof горутин**
```go
import _ "net/http/pprof"

go func() {
    http.ListenAndServe(":6060", nil)
}()
```

### **2. Stack trace при SIGQUIT**
```bash
kill -SIGQUIT <pid>
```

### **3. Логирование операций**
```go
func lockWithLog(mu *sync.Mutex, name string) {
    fmt.Printf("Горутина %d пытается захватить %s\n", getGoroutineID(), name)
    mu.Lock()
    fmt.Printf("Горутина %d захватила %s\n", getGoroutineID(), name)
}
```

## Итог

**Deadlock** — коварная ошибка, которую сложно обнаружить. Лучшая защита — профилактика:
- Единый порядок захвата ресурсов
- Таймауты на блокирующие операции
- Использование контекстов для отмены
- Тщательное проектирование concurrent логики
# 22 context

## Что такое Context

**Context** — это стандартизированный способ передачи метаданных, дедлайнов и сигналов отмены между горутинами.

## Основные цели

1. **Отмена операций** — уведомление дочерних горутин о необходимости завершения
2. **Таймауты** — автоматическая отмена операций по истечении времени
3. **Передача значений** — передача request-scoped данных
4. **Прерывание длительных операций**

## Базовый интерфейс

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

## Создание контекстов

### **1. Background & TODO**
```go
ctx := context.Background()  // базовый пустой контекст
ctx := context.TODO()        // когда не ясно какой контекст использовать
```

### **2. WithCancel - ручная отмена**
```go
ctx, cancel := context.WithCancel(context.Background())

go func() {
    <-ctx.Done()  // ждём сигнал отмены
    fmt.Println("Операция отменена")
}()

// Отменяем операцию
cancel()
```

### **3. WithTimeout - автоматическая отмена**
```go
// Автоматическая отмена через 5 секунд
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()  // освобождаем ресурсы

go func() {
    <-ctx.Done()
    if ctx.Err() == context.DeadlineExceeded {
        fmt.Println("Время вышло!")
    }
}()
```

### **4. WithDeadline - отмена к определённому времени**
```go
// Отмена в конкретный момент времени
deadline := time.Now().Add(2 * time.Hour)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()
```

### **5. WithValue - передача данных**
```go
// Передача request-scoped данных
type key string
const userKey key = "user"

ctx := context.WithValue(context.Background(), userKey, "Alice")

// Получение значения
if user, ok := ctx.Value(userKey).(string); ok {
    fmt.Println("User:", user)
}
```

## Практические примеры

### **HTTP сервер с таймаутом**
```go
func handler(w http.ResponseWriter, r *http.Request) {
    // Таймаут 3 секунды на обработку запроса
    ctx, cancel := context.WithTimeout(r.Context(), 3*time.Second)
    defer cancel()
    
    // Передаём контекст в базу данных
    result, err := db.QueryContext(ctx, "SELECT ...")
    if err != nil {
        if ctx.Err() == context.DeadlineExceeded {
            http.Error(w, "Request timeout", http.StatusGatewayTimeout)
            return
        }
    }
    // Обработка результата
}
```

### **Параллельная обработка с отменой**
```go
func processParallel(ctx context.Context, items []string) error {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    errCh := make(chan error, len(items))
    
    for _, item := range items {
        go func(item string) {
            select {
            case <-ctx.Done():
                return  // выходим если контекст отменён
            default:
                if err := processItem(item); err != nil {
                    errCh <- err
                    cancel()  // отменяем другие операции
                }
            }
        }(item)
    }
    
    // Ожидаем завершения или первой ошибки
    for range items {
        select {
        case err := <-errCh:
            return err
        case <-ctx.Done():
            return ctx.Err()
        }
    }
    
    return nil
}
```

## Правила использования

### **1. Передавайте контекст явно**
```go
// ХОРОШО
func GetUser(ctx context.Context, id int) (*User, error)

// ПЛОХО - скрытый контекст
func GetUser(id int) (*User, error)
```

### **2. Проверяйте контекст в циклах**
```go
func longOperation(ctx context.Context) error {
    for i := 0; i < 1000; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()  // выходим при отмене
        default:
            // выполняем работу
        }
    }
    return nil
}
```

### **3. Используйте в горутинах**
```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case job, ok := <-jobs:
            if !ok {
                return
            }
            process(job)
        case <-ctx.Done():
            return  // завершаемся при отмене контекста
        }
    }
}
```

## Особенности реализации

### **Древовидная структура**
```go
// Контексты образуют дерево
parent := context.Background()
child1, cancel1 := context.WithCancel(parent)
child2, cancel2 := context.WithTimeout(child1, time.Second)

// Отмена parent отменяет всех детей
// Отмена child1 отменяет child2
```

### **Value propagation**
```go
parent := context.WithValue(context.Background(), "key", "value")
child, _ := context.WithCancel(parent)

// Дочерний контекст наследует значения
val := child.Value("key")  // "value"
```

## Распространённые ошибки

### **1. Не проверяем Done()**
```go
// ОШИБКА
func badExample(ctx context.Context) {
    heavyOperation()  // может работать вечно
}

// ПРАВИЛЬНО
func goodExample(ctx context.Context) error {
    if err := ctx.Err(); err != nil {
        return err  // сразу выходим если контекст уже отменён
    }
    return heavyOperationWithContext(ctx)
}
```

### **2. Не вызываем cancel()**
```go
// УТЕЧКА ПАМЯТИ
ctx, cancel := context.WithCancel(context.Background())
// забыли defer cancel()

// ПРАВИЛЬНО
ctx, cancel := context.WithCancel(context.Background())
defer cancel()  // гарантированно освобождаем ресурсы
```

### **3. Хранение данных неправильных типов**
```go
// ОШИБКА - возможна коллизия ключей
ctx := context.WithValue(ctx, "userID", 123)

// ПРАВИЛЬНО - используем свой тип
type key string
const userIDKey key = "userID"
ctx := context.WithValue(ctx, userIDKey, 123)
```

## Производительность

- **WithCancel/WithTimeout** — очень легковесные операции
- **WithValue** — может быть медленнее при глубоких цепочках
- **Done() канал** — не блокирующая операция

## Итог

**Context** — essential инструмент для:
- Управления временем жизни операций
- Graceful shutdown приложений  
- Распространения метаданных в пределах запроса
- Создания отзывчивых и надёжных concurrent программ

Используйте контексты везде где есть блокирующие операции или возможна отмена.
# 23 ограничение capacity

## Capacity (ёмкость) — что это

**Capacity** — максимальное количество элементов, которое может вместить коллекция **без переаллокации памяти**.

## Где применяется capacity

### **1. Слайсы (slices)**
```go
// Создание слайса с capacity
slice := make([]int, 0, 100)  // length=0, capacity=100
```

### **2. Каналы (channels)**
```go
// Буферизованный канал
ch := make(chan int, 10)  // capacity=10
```

### **3. Мапы (maps)** — как hint
```go
// Предварительное выделение памяти
m := make(map[string]int, 1000)  // hint для начальной capacity
```

## Зачем ограничивать capacity

### **1. Экономия памяти**
```go
// ПЛОХО - много переаллокаций
var users []User
for i := 0; i < 10000; i++ {
    users = append(users, User{})  // многократные переаллокации
}

// ХОРОШО - заранее выделяем память
users := make([]User, 0, 10000)
for i := 0; i < 10000; i++ {
    users = append(users, User{})  // переаллокаций нет
}
```

### **2. Предотвращение копирования данных**
```go
data := make([]byte, 0, 1024)
for i := 0; i < 1000; i++ {
    data = append(data, byte(i))  // данные не копируются при append
}
```

### **3. Контроль памяти**
```go
// Ограничиваем максимальный размер
maxCapacity := 10000
data := make([]int, 0, maxCapacity)

// Защита от чрезмерного роста
if len(data) < maxCapacity {
    data = append(data, newItem)
}
```

## Правила роста capacity у слайсов

### **Стратегия увеличения**
```go
var s []int
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=0 cap=0

s = append(s, 1)
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=1 cap=1

s = append(s, 2)  
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=2 cap=2

s = append(s, 3)
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=3 cap=4 (удвоение)

s = append(s, 4, 5)
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=5 cap=8 (удвоение)
```

### **Алгоритм роста**
- До 1024 элементов: **удвоение** capacity
- После 1024 элементов: **+25%** каждый раз

## Практические примеры

### **Оптимизация парсинга**
```go
func parseLargeFile(data []byte) []string {
    // Заранее выделяем память под ожидаемое количество строк
    lines := make([]string, 0, countLinesHint(data))
    
    for _, line := range bytes.Split(data, []byte{'\n'}) {
        lines = append(lines, string(line))
    }
    return lines
}
```

### **Buffer pool с фиксированной capacity**
```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return bytes.NewBuffer(make([]byte, 0, 4096))  // фиксированный размер
    },
}

func getBuffer() *bytes.Buffer {
    buf := bufferPool.Get().(*bytes.Buffer)
    buf.Reset()  // сохраняем capacity, сбрасываем length
    return buf
}
```

## Ограничения и лимиты

### **1. Массивы — фиксированная capacity**
```go
arr := [100]int{}        // capacity всегда 100
// arr = append(arr, 101) // ОШИБКА: нельзя append к массиву
```

### **2. Максимальная capacity**
Теоретический максимум зависит от архитектуры:
- **32-битные системы**: ~2ГБ
- **64-битные системы**: огромные значения (на практике ограничены памятью)

### **3. Memory overhead**
```go
// Большая capacity ≠ большая память сразу
s := make([]int, 0, 1000000)  // выделяется память под 1M элементов
fmt.Println(len(s))            // но length = 0
```

## Управление capacity

### **1. Trim избыточной capacity**
```go
func processData(data []int) []int {
    result := make([]int, 0, len(data))
    
    for _, item := range data {
        if item > 0 {
            result = append(result, item)
        }
    }
    
    // Убираем лишнюю capacity если результат намного меньше
    if cap(result) > len(result)*2 {
        trimmed := make([]int, len(result))
        copy(trimmed, result)
        return trimmed
    }
    
    return result
}
```

### **2. Ограничение роста**
```go
type BoundedSlice struct {
    data []int
    maxCap int
}

func (bs *BoundedSlice) Append(item int) bool {
    if len(bs.data) >= bs.maxCap {
        return false  // превышен лимит
    }
    
    if len(bs.data) == cap(bs.data) {
        // Увеличиваем capacity, но не больше maxCap
        newCap := min(cap(bs.data)*2, bs.maxCap)
        newData := make([]int, len(bs.data), newCap)
        copy(newData, bs.data)
        bs.data = newData
    }
    
    bs.data = append(bs.data, item)
    return true
}
```

## Capacity vs Length

### **Length** — текущее количество элементов
### **Capacity** — максимальное количество без переаллокации

```go
s := make([]int, 5, 10)
fmt.Printf("len=%d cap=%d\n", len(s), cap(s))  // len=5 cap=10

// Можно добавить 5 элементов без переаллокации
for i := 0; i < 5; i++ {
    s = append(s, i)  // len=6..10, cap=10
}

// На 11-м элементе - переаллокация
s = append(s, 10)  // len=11, cap=20 (или больше)
```

## Best Practices

### **1. Указывайте capacity когда известен размер**
```go
// Известно точное количество
ids := make([]int, 0, len(userIDs))

// Известен примерный размер  
logs := make([]string, 0, 1000)  // ожидаем ~1000 записей
```

### **2. Не завышайте capacity без необходимости**
```go
// ПЛОХО - зря тратится память
huge := make([]byte, 0, 10*1024*1024)  // 10MB выделено, но не используется

// ХОРОШО - реалистичная оценка
reasonable := make([]byte, 0, 1024)  // 1KB достаточно
```

### **3. Используйте для предотвращения race conditions**
```go
func safeAppend(items []int, newItem int) []int {
    // Создаём новый слайс с достаточной capacity
    result := make([]int, len(items), len(items)+1)
    copy(result, items)
    result = append(result, newItem)
    return result
}
```

## Итог

**Capacity** — мощный инструмент оптимизации:
- ✅ **Ускоряет append операции** — меньше переаллокаций
- ✅ **Экономит память** — предотвращает временные пики
- ✅ **Улучшает производительность** — меньше копирований данных
- ⚠️ **Требует аккуратности** — можно зря потратить память

Используйте осознанно, когда известен или можно оценить требуемый размер данных.
# 24 slice с отрицательной длиной

## Короткий ответ: **НЕВОЗМОЖНО**

В Go **длина slice не может быть отрицательной**. При попытке создать slice с отрицательной длиной произойдёт **panic**.

## Примеры ошибок

### **1. Прямое создание с отрицательной длиной**
```go
// Это вызовет panic
s := make([]int, -1)
// panic: runtime error: makeslice: len out of range
```

### **2. Отрицательная длина через переменную**
```go
length := -5
s := make([]int, length) 
// panic: runtime error: makeslice: len out of range
```

### **3. Отрицательная capacity**
```go
// Capacity тоже не может быть отрицательной
s := make([]int, 0, -10)
// panic: runtime error: makeslice: cap out of range
```

## Почему так спроектировано

### **Семантика длинны в Go**
- **Length** — всегда неотрицательное целое число
- Отражает реальное количество элементов
- Отрицательная длина не имеет практического смысла

### **Защита от ошибок**
```go
// Вместо неопределённого поведения - явная ошибка
func createSlice(length int) []int {
    // Компилятор и runtime гарантируют, что length >= 0
    return make([]int, length)
}
```

## Обходные пути (но это антипаттерны)

### **1. Проверка перед созданием**
```go
func safeMakeSlice(length int) ([]int, error) {
    if length < 0 {
        return nil, fmt.Errorf("длина не может быть отрицательной: %d", length)
    }
    return make([]int, length), nil
}
```

### **2. Нормализация отрицательных значений**
```go
func normalizeSlice(length int) []int {
    if length < 0 {
        length = 0  // трактуем отрицательную длину как 0
    }
    return make([]int, length)
}
```

## Где могут появиться отрицательные значения

### **1. Вычисления с пользовательским вводом**
```go
func processUserInput(offset, limit int) []Data {
    length := limit - offset  // может стать отрицательным!
    
    if length < 0 {
        return []Data{}  // возвращаем пустой slice
    }
    
    return make([]Data, length)
}
```

### **2. Ошибки в алгоритмах**
```go
func calculateBatchSize(total, batch int) []Batch {
    remaining := total
    var batches []Batch
    
    for remaining > 0 {
        size := min(batch, remaining)
        // Если batch > total, size может стать отрицательным в следующей итерации
        batch := make([]Item, size)  // PANIC если size < 0
        batches = append(batches, Batch{Items: batch})
        remaining -= size
    }
    
    return batches
}
```

## Правильная обработка пограничных случаев

### **Защитное программирование**
```go
func createSliceSafely(length int) []int {
    switch {
    case length < 0:
        return nil  // или пустой slice: return []int{}
    case length == 0:
        return []int{}  // явно пустой
    default:
        return make([]int, length)
    }
}
```

### **Валидация входных данных**
```go
type SliceConfig struct {
    Length int `validate:"min=0"`
    Capacity int `validate:"min=0"`
}

func NewSlice(cfg SliceConfig) ([]int, error) {
    if cfg.Length < 0 || cfg.Capacity < 0 {
        return nil, errors.New("длина и capacity должны быть неотрицательными")
    }
    
    if cfg.Capacity < cfg.Length {
        cfg.Capacity = cfg.Length  // автоматическая коррекция
    }
    
    return make([]int, cfg.Length, cfg.Capacity), nil
}
```

## Сравнение с другими языками

### **Python**
```python
# В Python срезы с отрицательными индексами работают иначе
s = [1, 2, 3, 4, 5]
s[-1]  # 5 (последний элемент)
s[-2:] # [4, 5] (срез с конца)
```

### **JavaScript**
```javascript
// В JS отрицательная длина создаёт массив, но он ведёт себя странно
const arr = new Array(-1)  
console.log(arr.length) // RangeError: Invalid array length
```

## Практические рекомендации

### **1. Всегда проверяйте вычисляемые длины**
```go
func paginate(items []Item, page, pageSize int) []Item {
    start := page * pageSize
    end := start + pageSize
    
    if start < 0 { start = 0 }
    if end > len(items) { end = len(items) }
    if start >= end { return []Item{} }  // защита от отрицательной длины среза
    
    return items[start:end]
}
```

### **2. Используйте unsigned типы когда возможно**
```go
func createSlice(length uint) []int {
    // length гарантированно >= 0
    return make([]int, length)
}
```

### **3. Документируйте ожидания**
```go
// CreateSlice создает новый slice указанной длины.
// length должен быть неотрицательным числом, иначе произойдёт panic.
func CreateSlice(length int) []int {
    return make([]int, length)
}
```

## Итог

В Go **slice с отрицательной длиной невозможен** по дизайну языка. Это защитная мера, которая:
- ✅ **Предотвращает неопределённое поведение**
- ✅ **Делает код более предсказуемым**  
- ✅ **Обнаруживает ошибки на раннем этапе** (panic вместо тихого бага)

Всегда проверяйте значения, которые могут стать отрицательными, перед созданием slice.
# 25 slice с отрицательной  cap

## Короткий ответ: **НЕВОЗМОЖНО**

Как и с длиной, **capacity не может быть отрицательной**. Попытка создания slice с отрицательной capacity вызывает **panic**.

## Примеры ошибок

### **1. Прямое создание с отрицательной capacity**
```go
// Это вызовет panic
s := make([]int, 0, -5)
// panic: runtime error: makeslice: cap out of range
```

### **2. Отрицательная capacity через переменную**
```go
cap := -10
s := make([]int, 0, cap)
// panic: runtime error: makeslice: cap out of range
```

### **3. Оба параметра отрицательные**
```go
s := make([]int, -3, -5)
// panic: runtime error: makeslice: len out of range
// Проверка длины происходит раньше проверки capacity
```

## Почему так спроектировано

### **Логика capacity**
- **Capacity** — максимальное количество элементов без переаллокации
- Отрицательная capacity не имеет физического смысла
- Гарантирует целостность данных и предотвращает неопределённое поведение

### **Проверки в runtime**
```go
// Псевдокод из реализации Go
func makeslice(len, cap int) []byte {
    if len < 0 || cap < 0 || len > cap {
        panic("makeslice: len or cap out of range")
    }
    // ... выделение памяти
}
```

## Где могут возникнуть проблемы

### **1. Вычисления с пользовательским вводом**
```go
func createBuffer(initialSize, maxCapacity int) []byte {
    // Опасность: maxCapacity может быть отрицательным
    return make([]byte, initialSize, maxCapacity)  // PANIC!
}
```

### **2. Ошибки в бизнес-логике**
```go
func allocateMemory(dataSize, overhead int) []byte {
    totalCapacity := dataSize + overhead
    // Если overhead отрицательный и больше dataSize...
    return make([]byte, dataSize, totalCapacity)  // PANIC!
}
```

## Правильные подходы

### **1. Защитное программирование**
```go
func createSliceSafe(length, capacity int) ([]int, error) {
    if length < 0 || capacity < 0 {
        return nil, fmt.Errorf("length и capacity должны быть неотрицательными: len=%d, cap=%d", length, capacity)
    }
    if length > capacity {
        return nil, fmt.Errorf("length не может превышать capacity: len=%d, cap=%d", length, capacity)
    }
    return make([]int, length, capacity), nil
}
```

### **2. Автоматическая коррекция**
```go
func createSliceAuto(length, capacity int) []int {
    // Нормализуем отрицательные значения
    if length < 0 { length = 0 }
    if capacity < 0 { capacity = 0 }
    
    // Корректируем capacity если она меньше length
    if capacity < length {
        capacity = length
    }
    
    return make([]int, length, capacity)
}
```

### **3. Использование unsigned типов**
```go
func createSliceUnsigned(length uint, capacity uint) []int {
    // Гарантированно неотрицательные значения
    if capacity < length {
        capacity = length  // всё равно uint, не может быть отрицательным
    }
    return make([]int, int(length), int(capacity))
}
```

## Особые случаи

### **Slice expressions с отрицательной capacity?**
```go
arr := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

// В slice expressions нельзя указать отрицательную capacity напрямую
// Но можно использовать отрицательные индексы относительно длины массива
s1 := arr[2:7]    // len=5, cap=8 (от индекса 2 до конца массива)
s2 := arr[2:7:8]  // len=5, cap=6 (явно ограничили capacity)

// Отрицательные индексы работают только относительно длины slice/array
s3 := arr[5:5-2]  // ОШИБКА компиляции: invalid slice indices
```

## Практические примеры обработки

### **Конфигурируемый аллокатор**
```go
type SliceConfig struct {
    Length   int `json:"length" validate:"min=0"`
    Capacity int `json:"capacity" validate:"min=0"`
}

func NewSliceFromConfig(cfg SliceConfig) ([]byte, error) {
    if cfg.Length < 0 || cfg.Capacity < 0 {
        return nil, errors.New("negative dimensions not allowed")
    }
    
    if cfg.Capacity < cfg.Length {
        cfg.Capacity = cfg.Length  // автоматическая коррекция
    }
    
    return make([]byte, cfg.Length, cfg.Capacity), nil
}
```

### **Безопасные вычисления размера**
```go
func calculateSliceSize(requestedLen, requestedCap int) (int, int) {
    // Гарантируем неотрицательные значения
    length := max(0, requestedLen)
    capacity := max(0, requestedCap)
    
    // Гарантируем capacity >= length
    capacity = max(capacity, length)
    
    return length, capacity
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

## Отличие от других контейнеров

### **Каналы**
```go
// Каналы тоже не поддерживают отрицательную capacity
ch := make(chan int, -5)  // panic: negative buffer size
```

### **Maps**
```go
// Для map capacity - это только hint, не может быть отрицательной
m := make(map[string]int, -10)  // panic: negative map size
```

## Итог

**Slice с отрицательной capacity невозможен** в Go. Это осознанное дизайнерское решение, которое:

- ✅ **Предотвращает неопределённое поведение**
- ✅ **Обнаруживает ошибки на раннем этапе** (panic)
- ✅ **Сохраняет семантическую целостность** данных

**Всегда проверяйте и нормализуйте** значения перед созданием slice, особенно когда они приходят из внешних источников или вычисляются динамически.
