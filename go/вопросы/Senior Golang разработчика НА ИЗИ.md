# 1 - базовые типы данных
В Go (Golang) есть несколько базовых (встроенных) типов данных, которые можно разделить на следующие категории:

---

### 1. **Числовые типы**
#### Целые числа:
- **Знаковые** (могут быть отрицательными):  
  `int8`, `int16`, `int32`, `int64`  
  `int` — размер зависит от архитектуры (32 или 64 бита).
- **Беззнаковые** (только неотрицательные):  
  `uint8`, `uint16`, `uint32`, `uint64`, `uint`  
  `byte` — синоним для `uint8`.  
  `rune` — синоним для `int32` (хранит Unicode-символ).
- **Платформозависимые**:  
  `int` и `uint` — 32 или 64 бита в зависимости от ОС.

#### Числа с плавающей точкой:
- `float32` — 32-битное число (точность ~6 десятичных цифр).
- `float64` — 64-битное число (точность ~15 цифр, **используется по умолчанию**).

#### Комплексные числа:
- `complex64` (на основе `float32`).  
- `complex128` (на основе `float64`).  
Пример: `var c complex64 = 1 + 2i`.

---

### 2. **Строковый тип**
- `string` — неизменяемая последовательность байт (обычно в кодировке UTF-8).  
Пример: `s := "Hello, 世界"`.

---

### 3. **Логический тип**
- `bool` — принимает значения `true` или `false`.

---

### 4. **Специальные типы**
- **`byte`** — алиас для `uint8` (часто используется для бинарных данных).  
- **`rune`** — алиас для `int32` (хранит Unicode-символ, например `'A'` или `'世'`).  

---

### Примеры объявления переменных:
```go
var age int = 30
var price float64 = 29.99
var name string = "Alice"
var isActive bool = true
var symbol rune = '♥'
var data byte = 0xFF
```

---

### Особенности:
- **Нулевые значения**:  
  - Числа: `0`  
  - Строки: `""` (пустая строка)  
  - Логический тип: `false`  
- **Статическая типизация**: тип переменной нельзя изменить после объявления.
- **Вывод типов**: компилятор может определить тип автоматически:  
  ```go
  x := 42      // int
  y := 3.14    // float64
  z := "Hello" // string
  ```

---

### Строгая типизация
Go не позволяет смешивать типы в выражениях без явного преобразования:  
```go
var a int32 = 10
var b int64 = 20
// Ошибка: invalid operation (нельзя сложить int32 и int64)
// result := a + b // Так нельзя!
result := int64(a) + b // Корректно
```

Базовые типы в Go обеспечивают простоту, безопасность и эффективность для низкоуровневых операций.
# 2 - внутренне устройство строки

В Go строка имеет следующее внутреннее представление:

## Структура строки

```go
// Внутреннее представление (упрощенно)
type stringStruct struct {
    str unsafe.Pointer // указатель на массив байт
    len int           // длина строки в байтах
}
```

## Ключевые особенности

### 1. **Неизменяемость (immutable)**
```go
s := "Hello"
// s[0] = 'h' // Ошибка: нельзя изменять строку

// Создается НОВАЯ строка
s = "hello" + " world"
```

### 2. **UTF-8 кодировка по умолчанию**
```go
s := "Привет"  // автоматически в UTF-8
fmt.Println(len(s)) // 12 байт, а не 6 символов!
```

### 3. **Внутреннее хранение**
```go
s := "Hello"
// В памяти: | H | e | l | l | o |
//           ^
//           указатель str
// длина len = 5
```

## Примеры для понимания

### Длина в байтах vs символах
```go
s := "Hello, 世界"
fmt.Println(len(s)) // 13 байт
fmt.Println(utf8.RuneCountInString(s)) // 9 символов (рун)
```

### Эффективное копирование
```go
s1 := "very long string..."
s2 := s1 // Не копирует данные, только указатель!
```

## Преобразования

### Строка → Срез байт
```go
s := "Hello"
b := []byte(s) // Создает КОПИЮ данных
b[0] = 'h'
fmt.Println(s) // "Hello" (не изменилась)
```

### Срез байт → Строка
```go
b := []byte{'H', 'e', 'l', 'l', 'o'}
s := string(b) // Создает КОПИЮ данных
b[0] = 'h'
fmt.Println(s) // "Hello" (не изменилась)
```

## Оптимизации компилятора

### 1. **Пул строк (string interning)**
```go
s1 := "hello"
s2 := "hello"
// Компилятор может использовать один экземпляр в памяти
```

### 2. **Эффективные операции**
```go
// Компилятор оптимизирует конкатенацию
s := "a" + "b" + "c"
// Может быть преобразовано в однократное создание строки
```

## Практические следствия

### 1. **Эффективность подстрок**
```go
bigString := "very long text..."
sub := bigString[5:10] // Не копирует данные!
```

### 2. **Проблемы с большими строками**
```go
// Плохо: создает промежуточные копии
var result string
for i := 0; i < 1000; i++ {
    result += "data"
}

// Хорошо: использует strings.Builder
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString("data")
}
result := builder.String()
```

## Сравнение с другими языками

| Язык | Изменяемость | Кодировка |
|------|--------------|-----------|
| Go | Неизменяемая | UTF-8 |
| Java | Неизменяемая | UTF-16 |
| Python | Неизменяемая | UTF-8/16/32 |
| C++ | Изменяемая | Зависит от реализации |

Такое внутреннее устройство делает строки в Go эффективными для большинства операций, но требует осторожности при работе с большими объемами текста и частыми модификациями.
# 3 - длина строки

В Go есть два основных способа измерения длины строки, которые дают разные результаты:

## 1. **Длина в байтах (`len(string)`)**

Функция `len()` возвращает количество **байтов** в строке:

```go
package main

import "fmt"

func main() {
    s1 := "Hello"
    fmt.Println(len(s1)) // 5 (5 ASCII символов = 5 байт)
    
    s2 := "Привет"
    fmt.Println(len(s2)) // 12 (6 кириллических символов × 2 байта)
    
    s3 := "Hello, 世界"
    fmt.Println(len(s3)) // 13 (7 + 1 + 5 байт)
    // H e l l o , [пробел] = 7 байт
    // 世界 = 2 символа × 3 байта = 6 байт
}
```

## 2. **Длина в символах (рунах)**

Для подсчета количества Unicode символов (рун) используйте:

### Способ 1: `utf8.RuneCountInString()`
```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    s := "Hello, 世界"
    
    fmt.Println("Байты:", len(s))         // 13
    fmt.Println("Символы:", utf8.RuneCountInString(s)) // 9
}
```

### Способ 2: Преобразование в срез рун
```go
s := "Hello, 世界"
chars := []rune(s)
fmt.Println("Символы:", len(chars)) // 9
```

## Практические примеры

### Сравнение подходов
```go
texts := []string{
    "Hello",           // Только ASCII
    "Café",           // С акцентом
    "Привет",         // Кириллица  
    "Hello, 世界",     // Смешанный
    "😀🎉🌟",          // Эмодзи
}

for _, text := range texts {
    bytes := len(text)
    runes := utf8.RuneCountInString(text)
    fmt.Printf("Текст: %-10s Байты: %2d Символы: %2d\n", text, bytes, runes)
}
```

**Результат:**
```
Текст: Hello      Байты:  5 Символы:  5
Текст: Café       Байты:  5 Символы:  4
Текст: Привет     Байты: 12 Символы:  6
Текст: Hello, 世界 Байты: 13 Символы:  9
Текст: 😀🎉🌟      Байты: 12 Символы:  3
```

### Итерация по символам, а не байтам
```go
s := "Hello, 世界"

// ❌ Неправильно - итерация по байтам
for i := 0; i < len(s); i++ {
    fmt.Printf("%c ", s[i]) // Испорченные символы для многобайтовых
}

// ✅ Правильно - итерация по рунам
for _, char := range s {
    fmt.Printf("%c ", char) // Корректно отображает все символы
}
```

## Производительность

- **`len(string)`** - O(1) операция, так как длина хранится в структуре строки
- **`utf8.RuneCountInString()`** - O(n) операция, требует прохода по всей строке
- **`len([]rune(string))`** - O(n) + выделение памяти под срез

## Рекомендации

```go
// Для ASCII текста используйте len()
if len(username) < 3 {
    return errors.New("имя пользователя слишком короткое")
}

// Для Unicode текста используйте utf8.RuneCountInString()
if utf8.RuneCountInString(displayName) > 20 {
    return errors.New("отображаемое имя слишком длинное")
}

// Для проверки пустой строки оба способа эквивалентны
if len(s) == 0 {
    // или if utf8.RuneCountInString(s) == 0 {
    fmt.Println("Строка пустая")
}
```

**Вывод:** Всегда учитывайте разницу между длиной в байтах и длиной в символах, особенно при работе с Unicode текстом.
# 4 - Как устроены мапы в GO

Map (ассоциативный массив) в Go — это реализация хэш-таблицы, оптимизированная для производительности и памяти.

## Основная структура

Map состоит из двух основных компонентов:

**hmap** - заголовок map, содержащий метаданные:
- count - текущее количество элементов
- B - показатель размера (количество бакетов = 2^B)
- buckets - указатель на массив бакетов
- oldbuckets - указатель на старую таблицу во время роста
- hash0 - seed для хэш-функции

**bmap** - бакет (корзина), содержащий:
- tophash - массив из 8 верхних битов хэшей для быстрого поиска
- ключи и значения (обычно по 8 пар на бакет)
- overflow pointer - указатель на дополнительный бакет при переполнении

## Организация данных

Данные в map организованы следующим образом:

1. **Хэширование**: При вставке ключа вычисляется хэш-функция
2. **Определение бакета**: Младшие биты хэша определяют номер бакета
3. **Быстрый поиск**: В бакете сначала сравниваются tophash значения
4. **Точное сравнение**: При совпадении tophash сравниваются сами ключи

## Механизм роста

Map растет динамически по мере добавления элементов:

- **Условие роста**: Когда средняя нагрузка превышает 6.5 элементов на бакет
- **Процесс роста**: Создается новая таблица в 2 раза больше
- **Постепенная миграция**: Элементы переносятся постепенно при последующих операциях
- **Инкрементальность**: За одну операцию переносится один бакет

## Особенности производительности

- **Локализация памяти**: Ключи и значения хранятся раздельно внутри бакета для улучшения кэширования
- **Быстрый поиск**: tophash позволяет быстро отсечь несовпадающие ключи
- **Оптимизация для маленьких map**: Для map до 8 элементов используется специальная быстрая реализация

## Поведенческие особенности

1. **Случайный порядок итерации** - элементы возвращаются в произвольном порядке
2. **Нулевое значение** - неинициализированная map равна nil
3. **Несравнимость** - map нельзя сравнивать оператором ==
4. **Непотокобезопасность** - требуется синхронизация при конкурентном доступе

## Производительность операций

- Вставка: O(1) в среднем случае
- Поиск: O(1) в среднем случае  
- Удаление: O(1) в среднем случае
- Итерация: O(n) по всем элементам

## Рекомендации по использованию

- **Предварительное выделение**: Создавайте map с нужной емкостью через make(map[K]V, capacity)
- **Избегайте частого роста**: Каждый рост требует перераспределения памяти и перехэширования
- **Используйте указатели для больших значений**: Это уменьшает затраты на копирование

Map в Go представляет собой тщательно оптимизированную хэш-таблицу, которая обеспечивает высокую производительность для большинства сценариев использования, сочетая эффективное использование памяти с быстрым доступом к данным.
# 5 - Эвакуация в map Go

**Эвакуация** — это процесс постепенного перемещения элементов из старых бакетов в новые при увеличении размера map.

## Когда происходит эвакуация

Эвакуация запускается при достижении порога нагрузки:
- **Коэффициент нагрузки** = 6.5 элементов на бакет в среднем
- **Формула**: `count > 6.5 * 2^B`

## Процесс эвакуации

### 1. Подготовка новой таблицы
```go
// Создаем новые бакеты в 2 раза больше
newBuckets = makeBuckets(1 << (h.B + 1))
h.oldbuckets = h.buckets  // Сохраняем старые бакеты
h.buckets = newBuckets    // Переключаем на новые
h.B++                     // Увеличиваем показатель размера
```

### 2. Постепенное перемещение
Эвакуация происходит не сразу, а постепенно при каждой операции с map:

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    // Если map в процессе роста, эвакуируем один бакет
    if h.growing() {
        growWork(t, h, bucket)
    }
    // ... остальная логика вставки
}
```

### 3. Механизм growWork
```go
func growWork(t *maptype, h *hmap, bucket uintptr) {
    // Эвакуируем бакет, соответствующий текущей операции
    evacuate(t, h, bucket&h.oldbucketmask())
    
    // Дополнительно эвакуируем следующий бакет из очереди
    if h.growing() {
        evacuate(t, h, h.nevacuate)
    }
}
```

## Детали процесса evacuate

### Распределение элементов
При эвакуации каждый элемент может попасть в один из двух новых бакетов:

```
Старый бакет X делится на два новых:
- Новый бакет X (те же младшие биты)
- Новый бакет X + oldSize (дополнительный старший бит)
```

### Пример для B=2 → B=3
```
Старая таблица: 4 бакета (B=2)
Новая таблица: 8 бакетов (B=3)

Старый бакет 1 (01) делится на:
- Новый бакет 1 (001) - если дополнительный бит = 0
- Новый бакет 5 (101) - если дополнительный бит = 1
```

### Алгоритм перемещения
```go
func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
    b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
    
    // Создаем два целевых бакета в новой таблице
    var newBuckets [2]uintptr
    newBuckets[0] = newBucketBase
    newBuckets[1] = newBucketBase + oldBucketCount
    
    // Перемещаем каждый элемент
    for ; b != nil; b = b.overflow(t) {
        for i := 0; i < bucketCnt; i++ {
            if b.tophash[i] == empty {
                continue
            }
            
            // Определяем новый бакет на основе дополнительного бита хэша
            hash := keyHash(b.keys[i])
            newBucketIndex := (hash >> h.B) & 1  // 0 или 1
            
            // Перемещаем элемент в соответствующий новый бакет
            dst := newBuckets[newBucketIndex]
            // ... копирование ключа и значения
        }
    }
    
    // Помечаем старый бакет как эвакуированный
    h.nevacuate++
}
```

## Особенности реализации

### 1. **Инкрементальность**
- Эвакуация происходит постепенно
- За одну операцию перемещается 1-2 бакета
- Не блокирует работу с map на длительное время

### 2. **Сохранение производительности**
- Операции продолжают работать во время эвакуации
- Поиск проверяет оба расположения (старое и новое)
- Минимальные накладные расходы

### 3. **Финализация роста**
Когда все бакеты эвакуированы:
```go
h.oldbuckets = nil  // Освобождаем старую память
h.flags &^= sameSizeGrow
```

## Пример временной линии эвакуации

```
Время | Событие
------|--------
T0    | Map достигает порога роста
T0    | Создаются новые бакеты, oldbuckets = текущие
T1    | Операция #1: эвакуирует бакет 0
T2    | Операция #2: эвакуирует бакет 1
T3    | Операция #3: эвакуирует бакет 2
...   | ...
Tn    | Все бакеты эвакуированы, oldbuckets = nil
```

## Преимущества подхода

1. **Отсутствие резких пауз** - нет единого момента блокировки
2. **Распределение затрат** - стоимость роста распределена по многим операциям
3. **Продолжающаяся работа** - map остается доступной во время роста

Эвакуация делает рост map в Go эффективным и незаметным для пользователя, обеспечивая стабильную производительность даже при активном изменении размера.
# 6 - Интерфейсы/примеры использования

## Внутреннее устройство интерфейсов

### Структура интерфейса

В Go интерфейс представлен двумя компонентами:

```go
// Внутреннее представление (упрощенно)
type iface struct {
    tab  *itab          // таблица методов и информация о типе
    data unsafe.Pointer // указатель на конкретные данные
}

type itab struct {
    inter *interfacetype // информация о интерфейсе
    _type *_type         // информация о конкретном типе
    hash  uint32         // копия хэша типа
    _     [4]byte
    fun   [1]uintptr     // массив методов (гибкий)
}
```

### Пустой интерфейс
```go
type eface struct {
    _type *_type         // информация о типе
    data  unsafe.Pointer // указатель на данные
}
```

## Примеры использования интерфейсов

### 1. Базовый пример интерфейса

```go
package main

import (
    "fmt"
    "math"
)

// Определяем интерфейс
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Реализуем интерфейс для разных типов
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Функция, принимающая интерфейс
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

func main() {
    circle := Circle{Radius: 5}
    rectangle := Rectangle{Width: 10, Height: 5}
    
    PrintShapeInfo(circle)     // Area: 78.54, Perimeter: 31.42
    PrintShapeInfo(rectangle)  // Area: 50.00, Perimeter: 30.00
    
    // Слайс интерфейсов
    shapes := []Shape{circle, rectangle}
    for _, shape := range shapes {
        PrintShapeInfo(shape)
    }
}
```

### 2. Интерфейсы в стандартной библиотеке

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

// Использование стандартных интерфейсов
func processReader(r io.Reader) {
    data := make([]byte, 100)
    n, err := r.Read(data)
    if err != nil && err != io.EOF {
        panic(err)
    }
    fmt.Printf("Read %d bytes: %s\n", n, string(data[:n]))
}

func main() {
    // Разные типы, удовлетворяющие io.Reader
    stringReader := strings.NewReader("Hello from string reader")
    file, _ := os.Open("example.txt") // предположим, что файл существует
    defer file.Close()
    
    processReader(stringReader)
    processReader(file)
    
    // Также можно использовать bytes.Buffer, os.Stdin и др.
}
```

### 3. Композиция интерфейсов

```go
package main

import "fmt"

// Базовые интерфейсы
type Reader interface {
    Read() string
}

type Writer interface {
    Write(string)
}

// Композитный интерфейс
type ReadWriter interface {
    Reader
    Writer
}

// Тип, реализующий композитный интерфейс
type Console struct {
    buffer string
}

func (c *Console) Read() string {
    return c.buffer
}

func (c *Console) Write(s string) {
    c.buffer = s
    fmt.Println("Written to console:", s)
}

func Process(rw ReadWriter) {
    rw.Write("Hello World")
    fmt.Println("Read back:", rw.Read())
}

func main() {
    console := &Console{}
    Process(console)
}
```

### 4. Type assertions и type switches

```go
package main

import "fmt"

func describe(i interface{}) {
    // Type switch
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

func main() {
    var i interface{}
    
    i = 42
    describe(i)
    
    // Type assertion
    if value, ok := i.(int); ok {
        fmt.Printf("Successfully asserted as int: %d\n", value)
    }
    
    i = "hello"
    describe(i)
    
    i = true
    describe(i)
}
```

### 5. Практический пример: кэширование

```go
package main

import (
    "fmt"
    "time"
)

// Cache интерфейс
type Cache interface {
    Get(key string) (interface{}, bool)
    Set(key string, value interface{})
    Delete(key string)
}

// InMemoryCache реализация
type InMemoryCache struct {
    data map[string]interface{}
}

func NewInMemoryCache() *InMemoryCache {
    return &InMemoryCache{
        data: make(map[string]interface{}),
    }
}

func (c *InMemoryCache) Get(key string) (interface{}, bool) {
    value, exists := c.data[key]
    return value, exists
}

func (c *InMemoryCache) Set(key string, value interface{}) {
    c.data[key] = value
}

func (c *InMemoryCache) Delete(key string) {
    delete(c.data, key)
}

// TTLCache с временем жизни
type TTLCache struct {
    InMemoryCache
    ttl    time.Duration
    timers map[string]*time.Timer
}

func NewTTLCache(ttl time.Duration) *TTLCache {
    return &TTLCache{
        InMemoryCache: *NewInMemoryCache(),
        ttl:           ttl,
        timers:        make(map[string]*time.Timer),
    }
}

func (c *TTLCache) Set(key string, value interface{}) {
    c.InMemoryCache.Set(key, value)
    
    // Удаляем старый таймер если есть
    if timer, exists := c.timers[key]; exists {
        timer.Stop()
    }
    
    // Устанавливаем новый таймер
    c.timers[key] = time.AfterFunc(c.ttl, func() {
        c.Delete(key)
    })
}

func (c *TTLCache) Delete(key string) {
    c.InMemoryCache.Delete(key)
    if timer, exists := c.timers[key]; exists {
        timer.Stop()
        delete(c.timers, key)
    }
}

// Функция работающая с любым кэшем
func UseCache(cache Cache) {
    cache.Set("user:1", "John Doe")
    
    if value, exists := cache.Get("user:1"); exists {
        fmt.Println("Found:", value)
    }
    
    cache.Delete("user:1")
}

func main() {
    memoryCache := NewInMemoryCache()
    ttlCache := NewTTLCache(5 * time.Minute)
    
    UseCache(memoryCache)
    UseCache(ttlCache)
}
```

### 6. Mock-тестирование с интерфейсами

```go
package main

import "fmt"

// Database интерфейс для абстракции работы с БД
type Database interface {
    GetUser(id int) (string, error)
    SaveUser(id int, name string) error
}

// RealDatabase - реальная реализация
type RealDatabase struct{}

func (db *RealDatabase) GetUser(id int) (string, error) {
    // Реальная логика работы с БД
    return "Real User", nil
}

func (db *RealDatabase) SaveUser(id int, name string) error {
    // Реальная логика сохранения
    return nil
}

// MockDatabase для тестов
type MockDatabase struct {
    users map[int]string
}

func NewMockDatabase() *MockDatabase {
    return &MockDatabase{
        users: make(map[int]string),
    }
}

func (db *MockDatabase) GetUser(id int) (string, error) {
    if user, exists := db.users[id]; exists {
        return user, nil
    }
    return "", fmt.Errorf("user not found")
}

func (db *MockDatabase) SaveUser(id int, name string) error {
    db.users[id] = name
    return nil
}

// UserService использующий интерфейс
type UserService struct {
    db Database
}

func NewUserService(db Database) *UserService {
    return &UserService{db: db}
}

func (s *UserService) CreateUser(id int, name string) error {
    return s.db.SaveUser(id, name)
}

func (s *UserService) GetUserName(id int) (string, error) {
    return s.db.GetUser(id)
}

func main() {
    // В production используем реальную БД
    realDB := &RealDatabase{}
    userService := NewUserService(realDB)
    
    // В тестах используем mock
    mockDB := NewMockDatabase()
    testService := NewUserService(mockDB)
    
    _ = userService
    _ = testService
}
```

## Ключевые преимущества интерфейсов

1. **Полиморфизм**: Разные типы могут иметь одинаковое поведение
2. **Тестируемость**: Легко создавать mock-объекты для тестирования
3. **Гибкость**: Можно менять реализации без изменения кода клиента
4. **Декомпозиция**: Разделение абстракции и реализации

## Лучшие практики

- **Интерфейсы должны быть маленькими** (принцип разделения интерфейсов)
- **Принимайте интерфейсы, возвращайте структуры**
- **Используйте композицию для создания сложных интерфейсов**
- **Избегайте пустых интерфейсов, когда можно использовать типизированные**

Интерфейсы в Go обеспечивают мощный механизм для создания гибкого и поддерживаемого кода, позволяя эффективно разделять абстракции и реализации.
# 7 - Горутины
# 8 - i/o bound operations

## Что такое I/O Bound Operations?

**I/O Bound Operations** - это операции, производительность которых ограничена скоростью операций ввода-вывода, а не вычислительной мощностью CPU. Такие операции проводят большую часть времени в ожидании завершения I/O операций.

### Примеры I/O Bound операций:
- Сетевые запросы (HTTP, gRPC, базы данных)
- Чтение/запись файлов на диск
- Операции с базами данных
- Взаимодействие с внешними API
- Web scraping

## Особенности I/O Bound операций в Go

### Блокирующие vs Неблокирующие операции

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "time"
)

// Блокирующий I/O (плохой подход)
func blockingIO() {
    start := time.Now()
    
    // Последовательные HTTP запросы
    response1, _ := http.Get("https://api.example.com/data1")
    defer response1.Body.Close()
    
    response2, _ := http.Get("https://api.example.com/data2") 
    defer response2.Body.Close()
    
    response3, _ := http.Get("https://api.example.com/data3")
    defer response3.Body.Close()
    
    fmt.Printf("Blocking IO took: %v\n", time.Since(start))
}

// Неблокирующий I/O с горутинами (хороший подход)
func nonBlockingIO() {
    start := time.Now()
    var wg sync.WaitGroup
    
    urls := []string{
        "https://api.example.com/data1",
        "https://api.example.com/data2",
        "https://api.example.com/data3",
    }
    
    for _, url := range urls {
        wg.Add(1)
        go func(u string) {
            defer wg.Done()
            response, _ := http.Get(u)
            if response != nil {
                response.Body.Close()
            }
        }(url)
    }
    
    wg.Wait()
    fmt.Printf("Non-blocking IO took: %v\n", time.Since(start))
}
```

## Паттерны для работы с I/O Bound операциями

### 1. Worker Pool для I/O операций

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "sync"
)

type Job struct {
    URL string
}

type Result struct {
    URL  string
    Size int64
    Err  error
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    for job := range jobs {
        fmt.Printf("Worker %d processing %s\n", id, job.URL)
        
        resp, err := client.Get(job.URL)
        if err != nil {
            results <- Result{URL: job.URL, Err: err}
            continue
        }
        
        size, err := io.Copy(io.Discard, resp.Body)
        resp.Body.Close()
        
        results <- Result{
            URL:  job.URL,
            Size: size,
            Err:  err,
        }
    }
}

func main() {
    urls := []string{
        "https://httpbin.org/json",
        "https://httpbin.org/xml", 
        "https://httpbin.org/html",
        "https://httpbin.org/bytes/1024",
        "https://httpbin.org/bytes/2048",
    }
    
    const numWorkers = 3
    jobs := make(chan Job, len(urls))
    results := make(chan Result, len(urls))
    
    var wg sync.WaitGroup
    
    // Запускаем workers
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, jobs, results, &wg)
    }
    
    // Отправляем jobs
    for _, url := range urls {
        jobs <- Job{URL: url}
    }
    close(jobs)
    
    // Закрываем results после завершения всех workers
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Собираем результаты
    for result := range results {
        if result.Err != nil {
            fmt.Printf("Error fetching %s: %v\n", result.URL, result.Err)
        } else {
            fmt.Printf("Fetched %s: %d bytes\n", result.URL, result.Size)
        }
    }
}
```

### 2. Ограничение количества одновременных I/O операций

```go
package main

import (
    "context"
    "fmt"
    "golang.org/x/sync/semaphore"
    "net/http"
    "sync"
    "time"
)

func rateLimitedIO() {
    var (
        sem    = semaphore.NewWeighted(5) // Максимум 5 одновременных запросов
        client = &http.Client{Timeout: 10 * time.Second}
        wg     sync.WaitGroup
        mu     sync.Mutex
        results []string
    )
    
    urls := make([]string, 20)
    for i := range urls {
        urls[i] = fmt.Sprintf("https://httpbin.org/delay/%d", i%3+1)
    }
    
    ctx := context.Background()
    
    for _, url := range urls {
        wg.Add(1)
        go func(u string) {
            defer wg.Done()
            
            // Acquire semaphore
            if err := sem.Acquire(ctx, 1); err != nil {
                fmt.Printf("Failed to acquire semaphore: %v\n", err)
                return
            }
            defer sem.Release(1)
            
            // Выполняем HTTP запрос
            resp, err := client.Get(u)
            if err != nil {
                fmt.Printf("Error fetching %s: %v\n", u, err)
                return
            }
            defer resp.Body.Close()
            
            mu.Lock()
            results = append(results, fmt.Sprintf("Completed: %s", u))
            mu.Unlock()
        }(url)
    }
    
    wg.Wait()
    fmt.Printf("Completed %d requests\n", len(results))
}
```

### 3. Использование context для отмены I/O операций

```go
package main

import (
    "context"
    "fmt"
    "io"
    "net/http"
    "time"
)

func fetchWithTimeout(url string, timeout time.Duration) (string, error) {
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return "", err
    }
    
    client := http.DefaultClient
    resp, err := client.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return "", err
    }
    
    return string(body), nil
}

func main() {
    url := "https://httpbin.org/delay/3" // Сервер ждет 3 секунды
    
    // Пытаемся получить данные с таймаутом 2 секунды
    start := time.Now()
    data, err := fetchWithTimeout(url, 2*time.Second)
    
    if err != nil {
        fmt.Printf("Request failed after %v: %v\n", time.Since(start), err)
    } else {
        fmt.Printf("Request succeeded: %d bytes\n", len(data))
    }
}
```

### 4. Pipeline для обработки I/O данных

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "strings"
    "sync"
)

func processURLs(urls []string) {
    // Stage 1: Загрузка
    download := func(urls []string) <-chan string {
        out := make(chan string)
        go func() {
            defer close(out)
            client := http.Client{Timeout: 10 * time.Second}
            for _, url := range urls {
                resp, err := client.Get(url)
                if err != nil {
                    continue
                }
                body, _ := io.ReadAll(resp.Body)
                resp.Body.Close()
                out <- string(body)
            }
        }()
        return out
    }
    
    // Stage 2: Обработка
    process := func(in <-chan string) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for data := range in {
                // Пример обработки: подсчет слов
                words := len(strings.Fields(data))
                out <- words
            }
        }()
        return out
    }
    
    // Stage 3: Агрегация
    aggregate := func(in <-chan int) int {
        total := 0
        for count := range in {
            total += count
        }
        return total
    }
    
    // Запускаем pipeline
    downloaded := download(urls)
    processed := process(downloaded)
    result := aggregate(processed)
    
    fmt.Printf("Total words: %d\n", result)
}
```

## Оптимизации для I/O Bound операций

### 1. Connection Pooling

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
    "time"
)

func setupConnectionPool() *sql.DB {
    db, err := sql.Open("postgres", "user=postgres dbname=test sslmode=disable")
    if err != nil {
        panic(err)
    }
    
    // Настройка connection pool
    db.SetMaxOpenConns(25)       // Максимум одновременных соединений
    db.SetMaxIdleConns(25)       // Максимум idle соединений  
    db.SetConnMaxLifetime(5 * time.Minute) // Время жизни соединения
    
    return db
}

func concurrentDBQueries(db *sql.DB) {
    var wg sync.WaitGroup
    queries := []string{
        "SELECT * FROM users WHERE active = true",
        "SELECT COUNT(*) FROM orders",
        "SELECT * FROM products WHERE price > 100",
    }
    
    for i, query := range queries {
        wg.Add(1)
        go func(q string, id int) {
            defer wg.Done()
            
            start := time.Now()
            rows, err := db.Query(q)
            if err != nil {
                fmt.Printf("Query %d failed: %v\n", id, err)
                return
            }
            defer rows.Close()
            
            // Обработка результатов...
            fmt.Printf("Query %d completed in %v\n", id, time.Since(start))
        }(query, i)
    }
    
    wg.Wait()
}
```

### 2. Кэширование результатов I/O операций

```go
package main

import (
    "sync"
    "time"
)

type Cache struct {
    mu    sync.RWMutex
    data  map[string]cacheEntry
}

type cacheEntry struct {
    value      interface{}
    expiration time.Time
}

func NewCache(cleanupInterval time.Duration) *Cache {
    cache := &Cache{
        data: make(map[string]cacheEntry),
    }
    
    // Запускаем горутину для очистки просроченных записей
    go cache.cleanup(cleanupInterval)
    return cache
}

func (c *Cache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.data[key] = cacheEntry{
        value:      value,
        expiration: time.Now().Add(ttl),
    }
}

func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    entry, exists := c.data[key]
    if !exists || time.Now().After(entry.expiration) {
        return nil, false
    }
    
    return entry.value, true
}

func (c *Cache) cleanup(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    
    for range ticker.C {
        c.mu.Lock()
        now := time.Now()
        for key, entry := range c.data {
            if now.After(entry.expiration) {
                delete(c.data, key)
            }
        }
        c.mu.Unlock()
    }
}
```

### 3. Bulk Operations для уменьшения I/O

```go
package main

import (
    "fmt"
    "time"
)

// Вместо множества маленьких запросов - один большой
func bulkDatabaseOperations(records []Record) error {
    // Подготовка batch insert
    const batchSize = 100
    var batches [][]Record
    
    for batchSize < len(records) {
        records, batches = records[batchSize:], append(batches, records[0:batchSize:batchSize])
    }
    batches = append(batches, records)
    
    // Выполнение batch операций
    for i, batch := range batches {
        if err := insertBatch(batch); err != nil {
            return fmt.Errorf("batch %d failed: %w", i, err)
        }
    }
    
    return nil
}

func insertBatch(records []Record) error {
    // Реализация batch insert
    time.Sleep(10 * time.Millisecond) // Имитация I/O
    return nil
}

type Record struct {
    ID   int
    Data string
}
```

## Мониторинг и диагностика

### Профилирование I/O операций

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "runtime"
    "time"
)

func monitorIOBoundApp() {
    // Запускаем pprof сервер
    go func() {
        http.ListenAndServe(":6060", nil)
    }()
    
    // Мониторинг количества горутин
    go func() {
        for {
            time.Sleep(5 * time.Second)
            fmt.Printf("Goroutines: %d\n", runtime.NumGoroutine())
        }
    }()
}
```

## Лучшие практики для I/O Bound операций

1. **Используйте асинхронные операции** - не блокируйте главные горутины
2. **Ограничивайте параллелизм** - используйте semaphore или worker pools
3. **Используйте таймауты** - всегда устанавливайте разумные таймауты
4. **Кэшируйте результаты** - уменьшайте количество повторных I/O операций
5. **Используйте connection pooling** - для баз данных и HTTP клиентов
6. **Обрабатывайте ошибки** - I/O операции часто завершаются ошибками
7. **Мониторьте производительность** - используйте pprof и метрики

Правильная работа с I/O Bound операциями критически важна для создания высокопроизводительных приложений в Go. Использование горутин и каналов позволяет эффективно обрабатывать множество одновременных I/O операций без излишней нагрузки на систему.
# 9 - cpu bound operations

## Что такое CPU Bound Operations?

CPU Bound Operations (операции, ограниченные производительностью процессора) — это задачи, которые требуют интенсивных вычислений и максимально используют вычислительные ресурсы CPU. В отличие от I/O Bound операций, они проводят большую часть времени в вычислениях, а не в ожидании операций ввода-вывода.

## Ключевые характеристики CPU Bound операций:

- Высокая загрузка процессора
- Минимальное время ожидания I/O
- Производительность зависит от тактовой частоты и количества ядер
- Примеры: математические вычисления, обработка изображений, машинное обучение, шифрование

## Особенности работы с CPU Bound операциями в Go

### Проблема планировщика Go

Планировщик Go оптимизирован для I/O Bound операций. При работе с CPU Bound задачами возникают специфические проблемы:

1. **Чрезмерное переключение контекста** - слишком много горутин конкурируют за процессорное время
2. **Ложное разделение кэша (False Sharing)** - несколько ядер процессора работают с одной областью памяти
3. **Накладные расходы** - управление большим количеством горутин может съедать преимущества параллелизма

### Оптимальное количество горутин

Для CPU Bound задач оптимальное количество горутин равно количеству доступных CPU ядер:

```go
numGoroutines := runtime.NumCPU()
```

## Стратегии оптимизации

### 1. Worker Pool с ограничением

Создавайте пул воркеров, где количество горутин равно количеству ядер процессора:

```go
func worker(id int, jobs <-chan Task, results chan<- Result) {
    for task := range jobs {
        // Выполнение CPU интенсивной задачи
        result := processTask(task)
        results <- result
    }
}

// Инициализация пула
numWorkers := runtime.NumCPU()
for i := 0; i < numWorkers; i++ {
    go worker(i, jobs, results)
}
```

### 2. Привязка к потокам ОС

Для критически важных CPU Bound задач используйте привязку к потокам ОС:

```go
func criticalCPUWork() {
    runtime.LockOSThread()
    defer runtime.UnlockOSThread()
    
    // Критическая CPU Bound задача
    performIntensiveComputation()
}
```

### 3. Блочная обработка данных

Обрабатывайте данные блоками для лучшей кэш-локализации:

```go
func processDataBlock(data []float64, start, end int) {
    // Обработка блока данных
    for i := start; i < end; i++ {
        data[i] = expensiveComputation(data[i])
    }
}
```

## Техники оптимизации производительности

### 1. Уменьшение аллокаций памяти

Используйте пулы объектов и повторное использование памяти:

```go
var matrixPool = sync.Pool{
    New: func() interface{} {
        return make([][]float64, 1000)
    },
}

func getMatrix() [][]float64 {
    return matrixPool.Get().([][]float64)
}

func putMatrix(m [][]float64) {
    matrixPool.Put(m)
}
```

### 2. Кэш-дружественные алгоритмы

Организуйте данные для оптимального использования кэша процессора:

```go
// Плохо: случайный доступ к памяти
for i := 0; i < size; i++ {
    for j := 0; j < size; j++ {
        result[j][i] = matrix[i][j] // Плохая локализация
    }
}

// Хорошо: последовательный доступ к памяти
for i := 0; i < size; i++ {
    for j := 0; j < size; j++ {
        result[i][j] = matrix[i][j] // Хорошая локализация
    }
}
```

### 3. Векторизация вычислений

Обрабатывайте несколько элементов данных одновременно:

```go
func processBatch(data []float64) {
    // Обработка нескольких элементов в одной итерации
    for i := 0; i < len(data); i += 4 {
        processFourElements(data[i:])
    }
}
```

### 4. Предвычисление и мемоизация

Кэшируйте результаты дорогостоящих вычислений:

```go
var computationCache sync.Map

func expensiveComputation(input int) int {
    if val, ok := computationCache.Load(input); ok {
        return val.(int)
    }
    
    // Дорогое вычисление
    result := compute(input)
    computationCache.Store(input, result)
    return result
}
```

## Мониторинг и диагностика

### Профилирование CPU

Используйте встроенные инструменты профилирования:

```go
func startCPUProfile() {
    f, _ := os.Create("cpu.prof")
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    performCPUIntensiveWork()
}
```

### Анализ количества горутин

Следите за количеством горутин в системе:

```go
func monitorGoroutines() {
    for {
        count := runtime.NumGoroutine()
        if count > 1000 {
            log.Printf("Warning: high goroutine count: %d", count)
        }
        time.Sleep(10 * time.Second)
    }
}
```

## Практические рекомендации

### 1. Избегайте избыточного параллелизма

Не создавайте больше горутин, чем количество CPU ядер для CPU Bound задач:

```go
// Плохо
for i := 0; i < 1000; i++ {
    go cpuIntensiveTask() // Слишком много для CPU Bound
}

// Хорошо
numCPU := runtime.NumCPU()
for i := 0; i < numCPU; i++ {
    go cpuIntensiveTask()
}
```

### 2. Используйте appropriate granularity

Выбирайте правильный размер задач для баланса между накладными расходами и параллелизмом:

```go
// Слишком мелкие задачи - большие накладные расходы
func processTinyChunks() {
    for i := 0; i < 1000000; i++ {
        go processSingleItem(data[i]) // Слишком мелко
    }
}

// Слишком крупные задачи - недостаточный параллелизм
func processHugeChunk() {
    go processAllData(data) // Слишком крупно
}

// Правильный баланс
func processOptimalChunks() {
    chunkSize := len(data) / runtime.NumCPU()
    for i := 0; i < len(data); i += chunkSize {
        go processChunk(data[i:i+chunkSize])
    }
}
```

### 3. Минимизируйте синхронизацию

Уменьшайте использование мьютексов в горячих путях:

```go
// Плохо: частые блокировки
var mu sync.Mutex
func processWithLock() {
    mu.Lock()
    defer mu.Unlock()
    // Работа с shared data
}

// Хорошо: уменьшение contention
func processWithChannel() {
    results := make(chan Result)
    go func() {
        results <- computeResult()
    }()
    return <-results
}
```

## Распространенные ошибки

### 1. Неправильное использование горутин

```go
// Ошибка: создание горутин в цикле без ограничения
for item := range items {
    go process(item) // Может создать тысячи горутин
}

// Правильно: использование semaphore для ограничения
sem := make(chan struct{}, runtime.NumCPU())
for item := range items {
    sem <- struct{}{}
    go func(i Item) {
        defer func() { <-sem }()
        process(i)
    }(item)
}
```

### 2. Игнорирование локальности данных

```go
// Ошибка: плохая кэш-локализация
type Data struct {
    values [][]float64
}

func (d *Data) Process() {
    for col := 0; col < cols; col++ {
        for row := 0; row < rows; row++ {
            d.values[row][col] = compute() // Случайный доступ
        }
    }
}

// Правильно: последовательный доступ к памяти
func (d *Data) ProcessOptimized() {
    for row := 0; row < rows; row++ {
        for col := 0; col < cols; col++ {
            d.values[row][col] = compute() // Последовательный доступ
        }
    }
}
```

## Заключение

Оптимизация CPU Bound операций в Go требует понимания как работы планировщика Go, так и архитектуры современных процессоров. Ключевые принципы:

- Используйте количество горутин, равное количеству CPU ядер
- Оптимизируйте использование кэша процессора
- Минимизируйте накладные расходы на синхронизацию
- Профилируйте и измеряйте производительность
- Балансируйте между параллелизмом и накладными расходами

Правильно оптимизированные CPU Bound приложения в Go могут эффективно использовать многоядерные процессоры и достигать высокой производительности вычислений.
# 10 - стек горутины
# 11 - потоки
# 12 - мьютексы
# 13 - Принудительное завершение горутин
# 14 - контекст
# 15 - select
# 16 - виды контекст
# 17 - Примитивы Lock\RWlock
# 18 - каналы
# 19 - атомики
# 20 - REST
# 21 - grpc
# 22 - Аутентификация
# 23 - basic auth
# 24 - jwt
# 25 - Как отправить бинаркик через get-запрос
# 26 - Индексы в Postgres
# 27 - Оптимизация работы таблицы
# 28 -
# 29 -
# 30 -