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

## Что такое стек горутины?

**Стек горутины** — это область памяти, выделенная для хранения локальных переменных, параметров функций и информации о вызовах для конкретной горутины. В отличие от потоков операционной системы, горутины используют динамические стеки, которые могут расти и сокращаться по мере необходимости.

## Особенности стека горутины

### Начальный размер
- **2 КБ** — начальный размер стека каждой горутины
- Значительно меньше, чем у потоков ОС (обычно 1-8 МБ)

### Динамическое изменение
- Стек может автоматически расти при необходимости
- Может сокращаться при освобождении памяти
- Вся работа по управлению памятью стека прозрачна для программиста

## Внутреннее устройство

### Структура стека в памяти

```
Стек горутины:
+-------------------+  ← high addresses
|    аргументы      |
|   функции main    |
+-------------------+
|  возвращаемые     |
|     адреса        |
+-------------------+
|  сохраненные      |
|   регистры        |
+-------------------+
|  локальные        |
|   переменные      |
+-------------------+
|   кадры вызовов   |
|   других функций  |
+-------------------+  ← low addresses
|   guard page      |
+-------------------+
```

### Управление стеком в рантайме Go

Каждая горутина содержит информацию о своем стеке:

```go
type g struct {
    stack       stack   // [lo, hi) - границы стека
    stackguard0 uintptr // защитная страница для проверки переполнения
    stackguard1 uintptr // защитная страница для C-кода
    // ... другие поля
}

type stack struct {
    lo uintptr // нижняя граница
    hi uintptr // верхняя граница
}
```

## Механизм роста стека

### Когда стек растет

Стек автоматически увеличивается при:
1. **Нехватке места** для новых вызовов функций
2. **Выделении больших объектов** в стеке
3. **Глубокой рекурсии**

### Процесс роста стека

```go
// Псевдокод процесса роста стека
func growstack(gp *g) {
    // 1. Вычислить новый размер (обычно в 2 раза больше)
    oldsize := gp.stack.hi - gp.stack.lo
    newsize := oldsize * 2
    
    // 2. Проверить максимальный размер
    if newsize > maxstacksize {
        throw("stack overflow")
    }
    
    // 3. Выделить новый стек
    newstack := stackalloc(newsize)
    
    // 4. Скопировать данные из старого стека в новый
    copystack(gp, newstack)
    
    // 5. Освободить старый стек
    stackfree(gp.stack)
    
    // 6. Обновить указатели стека
    gp.stack = newstack
    gp.stackguard0 = newstack.lo + stackGuard
}
```

### Пример, демонстрирующий рост стека

```go
package main

import (
    "fmt"
    "runtime"
    "unsafe"
)

func printStackInfo() {
    var buf [1024]byte
    n := runtime.Stack(buf[:], false)
    fmt.Printf("Stack size: %d bytes\n", n)
}

func recursiveFunction(depth int) {
    var localArray [256]byte // Выделяем память в стеке
    
    if depth%100 == 0 {
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        fmt.Printf("Depth: %d, Stack alloc: %v\n", 
            depth, unsafe.Sizeof(localArray))
    }
    
    if depth > 0 {
        recursiveFunction(depth - 1)
    }
}

func main() {
    fmt.Printf("Initial goroutine count: %d\n", runtime.NumGoroutine())
    printStackInfo()
    
    // Вызываем рекурсивную функцию для демонстрации роста стека
    recursiveFunction(500)
    
    printStackInfo()
}
```

## Защита от переполнения стека

### Stack Guard

Go использует механизм "stack guard" для обнаружения переполнения стека:

- **StackGuard0** — защитная страница для Go-кода
- **StackGuard1** — защитная страница для C-кода (cgo)

При попытке доступа к защитной странице возникает паника с сообщением "stack overflow".

### Пример переполнения стека

```go
func causeStackOverflow() {
    var huge [1024 * 1024]byte // 1MB в стеке - может вызвать переполнение
    _ = huge
    causeStackOverflow() // Бесконечная рекурсия
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from:", r)
        }
    }()
    
    causeStackOverflow()
}
```

## Сравнение с традиционными потоками

### Потоки ОС
```go
// Традиционные потоки:
// - Фиксированный размер стека (1-8 МБ)
// - Большие накладные расходы на создание
// - Ограниченное количество (тысячи)
// - Переключение контекста дорогое
```

### Горутины Go
```go
// Горутины Go:
// - Динамический стек (2 КБ → ...)
// - Маленькие накладные расходы
// - Миллионы горутин возможны
// - Дешевое переключение
```

## Практические аспекты

### Мониторинг использования стека

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func monitorStackUsage() {
    for {
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        
        fmt.Printf("Goroutines: %d, Stack in use: %d KB\n",
            runtime.NumGoroutine(),
            m.StackInuse/1024)
        
        time.Sleep(2 * time.Second)
    }
}

func createManyGoroutines() {
    for i := 0; i < 1000; i++ {
        go func(id int) {
            // Каждая горутина использует свой стек
            var localData [512]byte
            _ = localData
            time.Sleep(time.Minute)
        }(i)
    }
}

func main() {
    go monitorStackUsage()
    createManyGoroutines()
    
    select {} // Бесконечное ожидание
}
```

### Эффективное использование стека

```go
// Плохо: большие массивы в стеке
func inefficient() {
    var largeArray [100000]byte // 100KB в стеке
    process(largeArray[:])
}

// Лучше: использование указателей или срезов
func efficient() {
    data := make([]byte, 100000) // В куче
    process(data)
}

// Или для маленьких данных в стеке
func optimized() {
    var smallArray [1024]byte // 1KB в стеке - нормально
    process(smallArray[:])
}
```

## Оптимизации компилятора

### Escape Analysis

Компилятор Go определяет, должны ли переменные размещаться в стеке или куче:

```go
func example() {
    x := 42 // Размещается в стеке (не убегает)
    y := make([]byte, 1024) // Может размещаться в стеке или куче
    
    // Если ссылка на y передается наружу, она "убегает" в кучу
    return y // y убегает в кучу
}

func noEscape() *int {
    x := 42
    return &x // x убегает в кучу - НЕ оптимизируется
}

func staysOnStack() int {
    x := 42
    return x // x остается в стеке
}
```

### Inlining и стек

```go
// Маленькие функции могут быть встроены (inlined)
func smallFunction(a, b int) int {
    return a + b
}

func caller() {
    result := smallFunction(10, 20)
    // После inlining эквивалентно:
    // result := 10 + 20
    // Экономятся вызовы и использование стека
}
```

## Производительность

### Бенчмарк создания горутин

```go
func BenchmarkGoroutineCreation(b *testing.B) {
    for i := 0; i < b.N; i++ {
        go func() {
            // Легковесная горутина
        }()
    }
}

func BenchmarkStackIntensive(b *testing.B) {
    for i := 0; i < b.N; i++ {
        stackHeavyOperation()
    }
}

func stackHeavyOperation() {
    var data [1024]int
    for i := range data {
        data[i] = i * i
    }
}
```

## Лучшие практики

1. **Избегайте глубокой рекурсии** — используйте итеративные алгоритмы
2. **Не создавайте гигантские массивы в стеке** — используйте кучу для больших данных
3. **Мониторьте использование стека** — особенно в долгоживущих приложениях
4. **Используйте буферизованные каналы** — для контроля над параллелизмом
5. **Профилируйте приложения** — используйте pprof для анализа использования стека

## Ограничения

- **Максимальный размер стека**: 1GB на 64-битных системах
- **Минимальный размер стека**: 2KB
- **Защитные страницы**: добавляют небольшие накладные расходы

Стек горутины — это ключевой компонент, который делает горутины легковесными и эффективными. Понимание его работы помогает писать более производительные и надежные Go-приложения.
# 11 - потоки

## Что такое потоки?

**Потоки** (threads) — это наименьшая единица выполнения в операционной системе. В контексте Go потоки — это потоки операционной системы, на которых планировщик Go выполняет горутины.

## Архитектура потоков в Go

### Модель M:P:G

Go использует гибридную модель планирования:

- **G (Goroutine)** — горутина, легковесный поток выполнения
- **M (Machine)** — поток операционной системы
- **P (Processor)** — контекст выполнения (логический процессор)

```
Runtime Go:
+---+---+---+
| P | P | P |  ← Processors (количество = GOMAXPROCS)
+---+---+---+
  |   |   |
  M   M   M   ← Threads (OS Threads)
  |   |   |
  G   G   G   ← Goroutines
  G   G   G
  .   .   .
```

### Структура потока (M) в Go

```go
// runtime/runtime2.go (упрощенно)
type m struct {
    g0      *g     // горутина для выполнения системных вызовов
    curg    *g     // текущая выполняемая горутина
    p       puintptr // привязанный P
    nextp   puintptr // следующий P
    oldp    puintptr // старый P (при syscall)
    
    // Связь с OS thread
    thread  uintptr // handle OS thread
    lockedg *g     // горутина, заблокировавшая этот thread
    
    // Системные вызовы
    syscall libcall // для системных вызовов
    
    // Локальный кэш
    mcache *mcache
}
```

## Управление потоками в Go

### Создание потоков

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func monitorThreads() {
    for {
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        
        // Количество горутин
        goroutines := runtime.NumGoroutine()
        
        // В Go нет прямой функции для получения количества потоков,
        // но можно оценить через профилирование или внешние инструменты
        
        fmt.Printf("Goroutines: %d\n", goroutines)
        time.Sleep(2 * time.Second)
    }
}

func main() {
    go monitorThreads()
    
    // Создаем множество горутин
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            time.Sleep(10 * time.Second)
        }(i)
    }
    
    wg.Wait()
}
```

### GOMAXPROCS - управление параллелизмом

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func demonstrateGOMAXPROCS() {
    // Получаем текущее значение
    current := runtime.GOMAXPROCS(0)
    fmt.Printf("Current GOMAXPROCS: %d\n", current)
    fmt.Printf("Number of CPU cores: %d\n", runtime.NumCPU())
    
    // Устанавливаем новое значение
    runtime.GOMAXPROCS(4)
    
    // Демонстрируем работу
    start := time.Now()
    
    var wg sync.WaitGroup
    for i := 0; i < 8; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            // CPU-intensive работа
            sum := 0
            for j := 0; j < 100000000; j++ {
                sum += j
            }
            fmt.Printf("Goroutine %d completed\n", id)
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("Work completed in %v\n", time.Since(start))
}
```

## Взаимодействие горутин и потоков

### Планирование горутин на потоках

```go
package main

import (
    "fmt"
    "runtime"
    "sync/atomic"
    "time"
)

func demonstrateScheduling() {
    var counter int32
    var wg sync.WaitGroup
    
    // Создаем горутины, которые будут показывать, на каком потоке выполняются
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            // Симулируем работу
            time.Sleep(100 * time.Millisecond)
            
            // Атомарно увеличиваем счетчик
            atomic.AddInt32(&counter, 1)
            
            fmt.Printf("Goroutine %d completed (total: %d)\n", id, atomic.LoadInt32(&counter))
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("All goroutines completed. Total: %d\n", counter)
}
```

### Системные вызовы и потоки

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

func demonstrateSyscalls() {
    var wg sync.WaitGroup
    urls := []string{
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/3",
    }
    
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    for i, url := range urls {
        wg.Add(1)
        go func(id int, u string) {
            defer wg.Done()
            
            start := time.Now()
            fmt.Printf("Goroutine %d starting HTTP request to %s\n", id, u)
            
            resp, err := client.Get(u)
            if err != nil {
                fmt.Printf("Goroutine %d error: %v\n", id, err)
                return
            }
            resp.Body.Close()
            
            duration := time.Since(start)
            fmt.Printf("Goroutine %d completed in %v (status: %d)\n", 
                id, duration, resp.StatusCode)
        }(i, url)
    }
    
    wg.Wait()
    fmt.Println("All HTTP requests completed")
}
```

## Специальные типы потоков в Go

### 1. Системный поток (g0)

Каждый поток M имеет специальную горутину g0, которая используется для:
- Запуска новых горутин
- Сбора мусора
- Управления стеком

### 2. Signal Handling Thread

Go создает специальный поток для обработки сигналов:
```go
// Этот поток обрабатывает:
// - SIGPROF (профилирование)
// - SIGURG (уведомления от сборщика мусора)
// - Другие POSIX-сигналы
```

## Блокирующие операции и потоки

### I/O операции и сетевой поллинг

```go
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

func demonstrateNetworkIO() {
    var wg sync.WaitGroup
    
    // Создаем TCP сервер
    listener, err := net.Listen("tcp", "localhost:0")
    if err != nil {
        panic(err)
    }
    defer listener.Close()
    
    // Запускаем сервер
    go func() {
        for {
            conn, err := listener.Accept()
            if err != nil {
                return
            }
            go handleConnection(conn)
        }
    }()
    
    // Создаем клиентов
    address := listener.Addr().String()
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(clientID int) {
            defer wg.Done()
            
            conn, err := net.Dial("tcp", address)
            if err != nil {
                fmt.Printf("Client %d: connection error: %v\n", clientID, err)
                return
            }
            defer conn.Close()
            
            message := fmt.Sprintf("Hello from client %d", clientID)
            conn.Write([]byte(message))
            
            buffer := make([]byte, 1024)
            n, _ := conn.Read(buffer)
            fmt.Printf("Client %d received: %s\n", clientID, string(buffer[:n]))
        }(i)
    }
    
    wg.Wait()
    time.Sleep(100 * time.Millisecond)
}

func handleConnection(conn net.Conn) {
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        return
    }
    
    response := fmt.Sprintf("Echo: %s", string(buffer[:n]))
    conn.Write([]byte(response))
}
```

### Системные вызовы и блокирующие операции

```go
package main

import (
    "fmt"
    "os"
    "sync"
    "time"
)

func demonstrateBlockingSyscalls() {
    var wg sync.WaitGroup
    
    // Создаем временные файлы
    filenames := make([]string, 5)
    for i := range filenames {
        file, err := os.CreateTemp("", "example")
        if err != nil {
            panic(err)
        }
        file.WriteString("Hello, World!")
        file.Close()
        filenames[i] = file.Name()
        defer os.Remove(file.Name())
    }
    
    // Читаем файлы параллельно (блокирующие системные вызовы)
    for i, filename := range filenames {
        wg.Add(1)
        go func(id int, fn string) {
            defer wg.Done()
            
            start := time.Now()
            
            // Блокирующий системный вызов
            data, err := os.ReadFile(fn)
            if err != nil {
                fmt.Printf("Goroutine %d error: %v\n", id, err)
                return
            }
            
            duration := time.Since(start)
            fmt.Printf("Goroutine %d read %d bytes in %v\n", 
                id, len(data), duration)
        }(i, filename)
    }
    
    wg.Wait()
}
```

## Управление потоком выполнения

### Runtime LockOSThread

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func demonstrateLockOSThread() {
    var wg sync.WaitGroup
    
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(threadID int) {
            // Привязываем горутину к текущему потоку ОС
            runtime.LockOSThread()
            defer runtime.UnlockOSThread()
            defer wg.Done()
            
            fmt.Printf("Goroutine %d locked to OS thread\n", threadID)
            
            // Долгая работа, которая должна выполняться на одном потоке
            for j := 0; j < 3; j++ {
                time.Sleep(500 * time.Millisecond)
                fmt.Printf("Goroutine %d working... (iteration %d)\n", threadID, j)
            }
            
            fmt.Printf("Goroutine %d completed\n", threadID)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("All locked goroutines completed")
}
```

## Мониторинг и отладка

### Профилирование потоков

```go
package main

import (
    "fmt"
    "log"
    "os"
    "runtime"
    "runtime/pprof"
    "time"
)

func profileThreads() {
    // Создаем файл для профилирования
    f, err := os.Create("thread_profile.prof")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    // Запускаем CPU профилирование
    if err := pprof.StartCPUProfile(f); err != nil {
        log.Fatal(err)
    }
    defer pprof.StopCPUProfile()
    
    // Выполняем работу
    performWork()
    
    // Создаем профиль горутин
    g, err := os.Create("goroutine_profile.pprof")
    if err != nil {
        log.Fatal(err)
    }
    defer g.Close()
    
    if err := pprof.Lookup("goroutine").WriteTo(g, 1); err != nil {
        log.Fatal(err)
    }
}

func performWork() {
    var wg sync.WaitGroup
    
    for i := 0; i < 50; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            // Смешанная нагрузка: CPU + I/O
            sum := 0
            for j := 0; j < 1000000; j++ {
                sum += j
            }
            
            time.Sleep(100 * time.Millisecond)
            
            _ = sum
        }(i)
    }
    
    wg.Wait()
}

func main() {
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("NumCPU: %d\n", runtime.NumCPU())
    
    profileThreads()
}
```

## Лучшие практики работы с потоками в Go

### 1. Настройка GOMAXPROCS

```go
func main() {
    // Автоматическая настройка
    runtime.GOMAXPROCS(runtime.NumCPU())
    
    // Или ручная настройка
    if os.Getenv("GOMAXPROCS") == "" {
        runtime.GOMAXPROCS(4)
    }
}
```

### 2. Избегание избыточного создания потоков

```go
func efficientWork() {
    // Используем worker pool вместо создания горутин для каждой задачи
    workers := runtime.NumCPU()
    jobs := make(chan Job, workers*2)
    results := make(chan Result, workers*2)
    
    // Запускаем фиксированное количество воркеров
    for i := 0; i < workers; i++ {
        go worker(i, jobs, results)
    }
}
```

### 3. Обработка блокирующих вызовов

```go
func handleBlockingCalls() {
    // Для блокирующих C вызовов используем отдельные горутины
    go func() {
        // Блокирующий вызов
        result := blockingCcall()
        // Обработка результата
    }()
}
```

## Проблемы и решения

### 1. Проблема: Чрезмерное создание потоков

**Решение**: Ограничение параллелизма и использование пулов

```go
func controlledConcurrency() {
    sem := make(chan struct{}, runtime.NumCPU()*2) // Семафор
    
    for i := 0; i < 100; i++ {
        sem <- struct{}{} // Захватываем слот
        go func(id int) {
            defer func() { <-sem }() // Освобождаем слот
            
            // Работа
            processItem(id)
        }(i)
    }
}
```

### 2. Проблема: False Sharing

**Решение**: Выравнивание данных

```go
type PaddedCounter struct {
    counter int64
    _       [56]byte // Заполнение до размера кэш-линии
}
```

## Заключение

Понимание работы потоков в Go критически важно для создания высокопроизводительных приложений. Ключевые моменты:

- Go использует гибридную модель M:P:G для планирования
- Количество потоков ОС обычно немного больше, чем GOMAXPROCS
- Планировщик Go эффективно управляет потоками автоматически
- Для особых случаев можно использовать `runtime.LockOSThread()`
- Мониторинг и профилирование помогают оптимизировать использование потоков

Правильное понимание и использование потоков позволяет создавать эффективные и масштабируемые приложения на Go.
# 12 - мьютексы

## Что такое мьютекс?

**Мьютекс** (mutual exclusion - взаимное исключение) — это примитив синхронизации, который обеспечивает эксклюзивный доступ к общим ресурсам в многопоточной среде. Только одна горутина может захватить мьютекс в любой момент времени.

## Основные концепции

### Состояния мьютекса
- **Заблокирован (Locked)** — мьютекс захвачен горутиной
- **Свободен (Unlocked)** — мьютекс доступен для захвата

### Базовые операции
- **Lock()** — захватить мьютекс (блокируется если уже занят)
- **Unlock()** — освободить мьютекс

## Типы мьютексов в Go

### 1. sync.Mutex - стандартный мьютекс

```go
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()        // Захватываем мьютекс
    counter++        // Критическая секция
    mu.Unlock()      // Освобождаем мьютекс
}
```

### 2. sync.RWMutex - читающе-писающий мьютекс

Позволяет множественное чтение или эксклюзивную запись:

```go
var rwMu sync.RWMutex
var data map[string]string

func readData(key string) string {
    rwMu.RLock()          // Захватываем для чтения
    defer rwMu.RUnlock()  // Гарантированно освобождаем
    return data[key]
}

func writeData(key, value string) {
    rwMu.Lock()           // Захватываем для записи
    defer rwMu.Unlock()   // Гарантированно освобождаем
    data[key] = value
}
```

## Принципы работы

### Как работает мьютекс

Когда горутина вызывает `Lock()`:
1. Если мьютекс свободен — захватывает его и продолжает выполнение
2. Если мьютекс занят — блокируется до его освобождения
3. При освобождении (`Unlock()`) одна из ожидающих горутин получает доступ

### Внутренняя реализация

Мьютекс в Go использует:
- **Атомарные операции** для быстрого пути (fast path)
- **Очередь ожидания** для заблокированных горутин
- **Алгоритм самоспиннинга** перед блокировкой

## Проблемы, которые решают мьютексы

### 1. Состояние гонки (Race Condition)
Когда несколько горутин одновременно обращаются к общим данным без синхронизации.

### 2. Атомарность операций
Обеспечивает, что сложные операции выполняются как единое целое.

### 3. Согласованность данных
Гарантирует, что другие горутины видят актуальное состояние данных.

## Лучшие практики

### 1. Всегда используйте defer для Unlock()

```go
func safeOperation() {
    mu.Lock()
    defer mu.Unlock() // Гарантированное освобождение
    // операции с общими данными
}
```

### 2. Минимизируйте время удержания мьютекса

```go
// Плохо
func slowOperation() {
    mu.Lock()
    // Долгие вычисления
    result = heavyCalculation()
    sharedData = result
    mu.Unlock()
}

// Хорошо
func fastOperation() {
    result := heavyCalculation() // Вычисления без блокировки
    mu.Lock()
    sharedData = result          // Только запись под блокировкой
    mu.Unlock()
}
```

### 3. Избегайте вложенных блокировок

```go
// Опасный код - может привести к deadlock
func nestedLocks() {
    mu1.Lock()
    mu2.Lock() // Если другая горутина сделает наоборот - deadlock
    // ...
    mu2.Unlock()
    mu1.Unlock()
}
```

## Распространенные ошибки

### 1. Забытый Unlock()
Приводит к deadlock - все остальные горутины ждут вечно.

### 2. Повторный Lock()
Попытка захватить уже захваченный мьютекс приводит к deadlock.

### 3. Использование копий мьютекса
Мьютексы нельзя копировать - нужно использовать указатели.

### 4. Неправильная область видимости
Мьютекс должен защищать данные, а не наоборот.

## Альтернативы мьютексам

### 1. Каналы
Для некоторых сценариев каналы могут быть более идиоматичным решением.

### 2. Atomic операции
Для простых счетчиков используйте `sync/atomic`.

### 3. sync.Map
Для определенных паттернов доступа к map.

## Производительность

Мьютексы в Go оптимизированы для:
- **Низкой конкуренции** — быстрый путь через атомарные операции
- **Высокой конкуренции** — эффективная очередь ожидания
- **Самоспиннинга** — кратковременное активное ожидание перед блокировкой

## Заключение

Мьютексы — фундаментальный инструмент синхронизации в Go. Понимание их работы и правильное использование критически важно для создания корректных конкурентных программ. Основные правила: защищайте мьютексом данные, а не код, минимизируйте время блокировки и всегда используйте defer для гарантированного освобождения.
# 13 - Принудительное завершение горутин

## Почему нельзя принудительно завершать горутины?

В Go **нет механизма** для принудительного завершения горутин извне. Это сознательное дизайнерское решение, основанное на нескольких принципах:

### Причины отсутствия принудительного завершения:
- **Безопасность памяти** - риск оставить данные в некорректном состоянии
- **Утечки ресурсов** - незакрытые файлы, сетевые соединения, мьютексы
- **Непредсказуемость** - сложно гарантировать корректность работы

## Правильные подходы к завершению горутин

### 1. Паттерн отмены через канал

```go
done := make(chan struct{})
go func() {
    for {
        select {
        case <-done:  // Проверяем сигнал отмены
            return    // Корректно завершаемся
        default:
            // Работа
        }
    }
}()

// Завершаем горутину
close(done)
```

### 2. Использование Context

```go
ctx, cancel := context.WithCancel(context.Background())

go func(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():  // Получаем сигнал отмены
            // Корректно освобождаем ресурсы
            cleanup()
            return
        default:
            performWork()
        }
    }
}(ctx)

// Завершаем все горутины, использующие этот контекст
cancel()
```

### 3. Таймауты и дедлайны

```go
// Горутина автоматически завершится через 5 секунд
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

go worker(ctx)
```

## Проблемные сценарии и решения

### 1. Зависшие горутины

**Проблема**: Горутина заблокирована на операции, которая никогда не завершится.

**Решение**: Использование таймаутов:
```go
select {
case result := <-operationChan:
    // Обрабатываем результат
case <-time.After(5 * time.Second):
    // Таймаут - выходим из горутины
    return
}
```

### 2. Горутины, игнорирующие сигналы отмены

**Проблема**: Горутина не проверяет каналы отмены.

**Решение**: Структурировать код для периодической проверки:
```go
func worker(ctx context.Context) {
    for i := 0; i < 1000; i++ {
        // Периодически проверяем контекст
        if i%10 == 0 {
            select {
            case <-ctx.Done():
                return
            default:
            }
        }
        // Работа
    }
}
```

## Опасные обходные пути

### 1. Runtime.Goexit (не рекомендуется)

```go
go func() {
    defer fmt.Println("Завершение")
    runtime.Goexit()  // Немедленно завершает текущую горутину
}()
```

**Проблемы**: Может привести к утечкам ресурсов, незавершенным операциям.

### 2. Паника в горутине

```go
go func() {
    defer func() {
        if r := recover(); r != nil {
            // Обработка паники
        }
    }()
    panic("принудительное завершение")
}()
```

**Проблемы**: Неструктурированное завершение, сложность отладки.

## Лучшие практики

### 1. Всегда используйте defer для очистки
```go
go func() {
    resource := acquireResource()
    defer releaseResource(resource)  // Гарантированная очистка
    
    // Работа с ресурсом
}()
```

### 2. Проектируйте горутины с возможностью завершения
```go
type Worker struct {
    quit chan struct{}
}

func (w *Worker) Stop() {
    close(w.quit)  // Сигнал завершения
}
```

### 3. Мониторинг завершения
```go
var wg sync.WaitGroup
wg.Add(1)

go func() {
    defer wg.Done()
    // Работа горутины
}()

// Ждем завершения
wg.Wait()
```

## Исключения: когда принудительное завершение оправдано

### 1. Тестовые сценарии
```go
func TestTimeout(t *testing.T) {
    go func() {
        time.Sleep(10 * time.Second)  // Долгая операция
    }()
    
    // В тестах иногда используют таймауты
    select {
    case <-time.After(1 * time.Second):
        t.Fatal("тест превысил время выполнения")
    }
}
```

### 2. Критические ошибки
```go
go func() {
    if criticalError {
        log.Fatal("критическая ошибка")  // Завершает программу
    }
}()
```

## Заключение

В Go принудительное завершение горутин считается антипаттерном. Вместо этого используйте:

- **Каналы отмены** для сигнализации о необходимости завершения
- **Context** для распространения сигналов отмены
- **Таймауты** для ограничения времени выполнения
- **WaitGroup** для ожидания корректного завершения

Такой подход обеспечивает надежность, предсказуемость и безопасность ваших concurrent-программ.
# 14 - контекст

## Что такое Context?

**Context** (контекст) — это стандартный механизм в Go для передачи метаданных, сигналов отмены и дедлайнов между горутинами. Это один из самых важных инструментов для управления жизненным циклом горутин.

## Основные концепции

### Назначение Context:
- **Передача сигналов отмены** между горутинами
- **Установка таймаутов** и дедлайнов
- **Передача значений** (request-scoped данных)
- **Координация работы** нескольких горутин

## Типы Context

### 1. Background Context
```go
ctx := context.Background() // Базовый контекст, никогда не отменяется
```

### 2. TODO Context
```go
ctx := context.TODO() // Заглушка, когда не ясно какой контекст использовать
```

### 3. Производные контексты

#### WithCancel - с возможностью отмены
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel() // Важно вызывать cancel для освобождения ресурсов

// Где-то в коде: cancel() - отменяет все производные контексты
```

#### WithTimeout - с таймаутом
```go
// Автоматическая отмена через 5 секунд
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

#### WithDeadline - с абсолютным временем
```go
// Автоматическая отмена в конкретное время
deadline := time.Now().Add(2 * time.Hour)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()
```

#### WithValue - для передачи значений
```go
type keyType string
const userKey keyType = "user"

ctx := context.WithValue(context.Background(), userKey, "Alice")
```

## Практическое использование

### Базовый шаблон работы с контекстом
```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done(): // Проверяем отмену контекста
            fmt.Println("Worker stopped:", ctx.Err())
            return
        default:
            // Выполняем работу
            performTask()
        }
    }
}
```

### Пример с HTTP запросом
```go
func httpCallWithTimeout(ctx context.Context, url string) (string, error) {
    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    client := &http.Client{}
    
    resp, err := client.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    return string(body), nil
}

// Использование
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

result, err := httpCallWithTimeout(ctx, "https://api.example.com")
```

## Методы Context интерфейса

### Deadline() 
Возвращает время, когда контекст будет автоматически отменен
```go
if deadline, ok := ctx.Deadline(); ok {
    fmt.Println("Context will expire at:", deadline)
}
```

### Done()
Возвращает канал, который закрывается при отмене контекста
```go
select {
case <-ctx.Done():
    return ctx.Err() // Context canceled или Deadline exceeded
}
```

### Err()
Возвращает ошибку отмены контекста
```go
if err := ctx.Err(); err != nil {
    switch err {
    case context.Canceled:
        fmt.Println("Context was canceled")
    case context.DeadlineExceeded:
        fmt.Println("Context deadline exceeded")
    }
}
```

### Value(key)
Получение значения из контекста
```go
if userID, ok := ctx.Value("userID").(string); ok {
    fmt.Println("User ID:", userID)
}
```

## Best Practices

### 1. Передавайте Context как первый параметр
```go
// Правильно
func ProcessData(ctx context.Context, data []byte) error

// Неправильно  
func ProcessData(data []byte, ctx context.Context) error
```

### 2. Всегда проверяйте Context в циклах
```go
func longRunningTask(ctx context.Context) error {
    for i := 0; i < 1000; i++ {
        // Периодически проверяем контекст
        if i%100 == 0 {
            select {
            case <-ctx.Done():
                return ctx.Err()
            default:
            }
        }
        // Работа
    }
    return nil
}
```

### 3. Используйте defer cancel()
```go
func operation() {
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel() // Гарантированно освобождаем ресурсы
    
    // Используем ctx
}
```

### 4. Вложенные контексты
```go
func handleRequest(parentCtx context.Context) {
    // Создаем дочерний контекст с более строгим таймаутом
    ctx, cancel := context.WithTimeout(parentCtx, 100*time.Millisecond)
    defer cancel()
    
    processData(ctx)
}
```

## Распространенные ошибки

### 1. Игнорирование контекста
```go
// Плохо - функция игнорирует переданный контекст
func badFunction(ctx context.Context) {
    time.Sleep(10 * time.Second) // Не проверяет отмену контекста
}
```

### 2. Неправильное использование WithValue
```go
// Плохо - использование базовых типов как ключей
ctx := context.WithValue(ctx, "userID", 123)

// Хорошо - использование пользовательских типов
type contextKey string
const userIDKey contextKey = "userID"
ctx := context.WithValue(ctx, userIDKey, 123)
```

### 3. Утечка горутин
```go
// Плохо - горутина может работать вечно
go func() {
    for {
        // работа без проверки контекста
    }
}()

// Хорошо - горутина слушает контекст
go func(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            // работа
        }
    }
}(ctx)
```

## Пример полного приложения

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func databaseQuery(ctx context.Context, query string) (string, error) {
    // Имитация долгого запроса к БД
    select {
    case <-time.After(2 * time.Second):
        return "query result", nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

func apiCall(ctx context.Context, url string) (string, error) {
    // Имитация API вызова
    select {
    case <-time.After(1 * time.Second):
        return "api response", nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

func processUserRequest(ctx context.Context, userID string) error {
    // Добавляем userID в контекст
    ctx = context.WithValue(ctx, "userID", userID)
    
    // Параллельные операции с общим таймаутом
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    // Запускаем параллельные операции
    resultCh := make(chan string, 2)
    errCh := make(chan error, 2)
    
    go func() {
        result, err := databaseQuery(ctx, "SELECT * FROM users")
        if err != nil {
            errCh <- err
            return
        }
        resultCh <- result
    }()
    
    go func() {
        result, err := apiCall(ctx, "https://api.example.com/user")
        if err != nil {
            errCh <- err
            return
        }
        resultCh <- result
    }()
    
    // Ожидаем результаты
    for i := 0; i < 2; i++ {
        select {
        case result := <-resultCh:
            fmt.Println("Received:", result)
        case err := <-errCh:
            return err
        case <-ctx.Done():
            return ctx.Err()
        }
    }
    
    return nil
}

func main() {
    ctx := context.Background()
    
    if err := processUserRequest(ctx, "user123"); err != nil {
        fmt.Println("Error:", err)
    }
}
```

## Когда использовать Context

### Обязательно:
- **HTTP handlers** - используйте `r.Context()`
- **GRPC** - встроенная поддержка контекста
- **Фоновые задачи** с возможностью отмены
- **Работа с внешними API** и базами данных

### Рекомендуется:
- **Любые долгие операции**
- **Параллельная обработка** данных
- **Сервисы** с возможностью graceful shutdown

## Заключение

Context — это мощный инструмент для управления жизненным циклом операций в Go. Правильное использование контекста позволяет:

- Эффективно управлять ресурсами
- Обеспечивать отзывчивость приложений
- Координировать работу множества горутин
- Реализовывать graceful shutdown

Контекст стал стандартом де-факто для передачи метаданных и сигналов отмены в современных Go-приложениях.
# 15 - select

## Что такое select?

**Select** — это конструкция в Go, которая позволяет горутине ожидать выполнения нескольких операций с каналами одновременно. Это мощный инструмент для работы с конкурентностью, похожий на switch, но для каналов.

## Базовый синтаксис

```go
select {
case <-channel1:
    // обработка данных из channel1
case data := <-channel2:
    // обработка данных из channel2
case channel3 <- value:
    // отправка успешна
default:
    // выполняется, если ни один канал не готов
}
```

## Основные возможности

### 1. Ожидание нескольких каналов
```go
ch1 := make(chan string)
ch2 := make(chan int)

go func() {
    time.Sleep(1 * time.Second)
    ch1 <- "hello"
}()

go func() {
    time.Sleep(2 * time.Second) 
    ch2 <- 42
}()

select {
case msg := <-ch1:
    fmt.Println("Received:", msg)
case num := <-ch2:
    fmt.Println("Received:", num)
}
```

### 2. Таймауты
```go
select {
case result := <-longOperation():
    fmt.Println("Operation completed:", result)
case <-time.After(3 * time.Second):
    fmt.Println("Operation timed out")
}
```

### 3. Non-blocking операции
```go
select {
case value := <-channel:
    fmt.Println("Received:", value)
default:
    fmt.Println("No value ready") // Не блокируется
}
```

## Практические примеры

### Ожидание с таймаутом
```go
func waitWithTimeout() {
    done := make(chan bool)
    
    go func() {
        time.Sleep(5 * time.Second) // Долгая операция
        done <- true
    }()
    
    select {
    case <-done:
        fmt.Println("Operation completed")
    case <-time.After(3 * time.Second):
        fmt.Println("Timeout exceeded")
    }
}
```

### Приоритетная обработка
```go
func prioritySelect() {
    urgent := make(chan string)
    normal := make(chan string)
    
    // Сначала проверяем срочные сообщения
    select {
    case msg := <-urgent:
        fmt.Println("Urgent:", msg)
    case msg := <-normal:
        fmt.Println("Normal:", msg) 
    default:
        fmt.Println("No messages")
    }
}
```

### Multiple select в цикле
```go
func worker(stopCh <-chan struct{}, dataCh <-chan int) {
    for {
        select {
        case data := <-dataCh:
            fmt.Println("Processing:", data)
        case <-stopCh:
            fmt.Println("Worker stopping")
            return
        }
    }
}
```

## Особенности работы

### Случайный выбор при множестве готовых каналов
```go
ch1 := make(chan int, 1)
ch2 := make(chan int, 1)

ch1 <- 1
ch2 <- 2

// Выбор случая будет случайным
select {
case v := <-ch1:
    fmt.Println("From ch1:", v)
case v := <-ch2:
    fmt.Println("From ch2:", v)
}
```

### Select с отправкой в каналы
```go
func sendWithBackpressure() {
    ch := make(chan int, 1)
    
    // Пытаемся отправить без блокировки
    select {
    case ch <- 42:
        fmt.Println("Sent successfully")
    default:
        fmt.Println("Channel full, skipping")
    }
}
```

## Паттерны использования

### 1. Graceful shutdown
```go
func server(shutdownCh <-chan struct{}, requests <-chan Request) {
    for {
        select {
        case req := <-requests:
            handleRequest(req)
        case <-shutdownCh:
            cleanup()
            return
        }
    }
}
```

### 2. Ограничение времени выполнения
```go
func operationWithDeadline() error {
    resultCh := make(chan string)
    
    go func() {
        result := expensiveOperation()
        resultCh <- result
    }()
    
    select {
    case result := <-resultCh:
        fmt.Println("Success:", result)
        return nil
    case <-time.After(5 * time.Second):
        return errors.New("operation timed out")
    }
}
```

### 3. Периодические задачи
```go
func periodicWorker() {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            performPeriodicTask()
        case <-stopCh:
            return
        }
    }
}
```

## Обработка закрытых каналов

### Обнаружение закрытия канала
```go
func readUntilClosed(ch <-chan int) {
    for {
        select {
        case value, ok := <-ch:
            if !ok {
                fmt.Println("Channel closed")
                return
            }
            fmt.Println("Received:", value)
        }
    }
}
```

## Продвинутые техники

### Select с контекстом
```go
func operationWithContext(ctx context.Context, ch <-chan int) {
    for {
        select {
        case data := <-ch:
            process(data)
        case <-ctx.Done():
            fmt.Println("Context canceled:", ctx.Err())
            return
        }
    }
}
```

### Мультиплексирование каналов
```go
func fanIn(inputs ...<-chan int) <-chan int {
    output := make(chan int)
    
    for _, input := range inputs {
        go func(ch <-chan int) {
            for value := range ch {
                output <- value
            }
        }(input)
    }
    
    return output
}

// Использование
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    
    multiplexed := fanIn(ch1, ch2)
    
    for value := range multiplexed {
        fmt.Println("Received:", value)
    }
}
```

## Распространенные ошибки

### 1. Deadlock с пустым select
```go
select {} // Вечный deadlock
```

### 2. Игнорирование закрытых каналов
```go
select {
case value := <-ch: // Может получать нулевые значения после закрытия
    // ...
}
```

### 3. Неправильная работа с default
```go
// Плохо - может создать busy loop
for {
    select {
    case data := <-ch:
        process(data)
    default:
        // Пустой default создает активное ожидание
    }
}

// Лучше - добавить небольшую паузу
for {
    select {
    case data := <-ch:
        process(data)
    default:
        time.Sleep(10 * time.Millisecond)
    }
}
```

## Best Practices

### 1. Всегда обрабатывайте все случаи
```go
select {
case result := <-resultCh:
    handleResult(result)
case err := <-errorCh:
    handleError(err)
case <-time.After(timeout):
    handleTimeout()
}
```

### 2. Используйте default для non-blocking операций
```go
select {
case ch <- value:
    // отправлено
default:
    // не блокируемся если канал полный
}
```

### 3. Комбинируйте с контекстом для отмены
```go
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-operation():
    return result
}
```

### 4. Избегайте вложенных select
```go
// Плохо
select {
case a := <-ch1:
    select {
    case b := <-ch2:
        // сложная логика
    }
}

// Лучше - используйте отдельные функции
```

## Производительность

Select в Go оптимизирован и эффективен:
- **O(1)** сложность для небольшого количества случаев
- **Случайный выбор** предотвращает голодание
- **Минимальные накладные расходы** по сравнению с другими подходами

## Заключение

Select — это фундаментальный инструмент для работы с конкурентностью в Go. Он позволяет:
- Координировать работу нескольких горутин
- Реализовывать таймауты и дедлайны
- Создавать неблокирующие операции
- Эффективно мультиплексировать каналы

Правильное использование select делает Go-программы более отзывчивыми, надежными и эффективными.
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