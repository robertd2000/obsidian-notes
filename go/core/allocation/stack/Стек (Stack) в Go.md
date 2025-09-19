Расскажу подробно о стеке и куче в Go с техническими деталями.

# Стек (Stack) в Go

## Что такое стек

**Стек** - это область памяти, которая работает по принципу LIFO (Last-In, First-Out). Каждая горутина в Go имеет свой собственный стек.

## Технические характеристики стека

### 1. Организация памяти

```
+-------------------+ ← Высокие адреса
|       ...         |
+-------------------+
|   Аргументы       | ← Caller's frame
|   функции         |
+-------------------+
|   Return address  |
+-------------------+
|   Saved BP        | ← Base pointer
+-------------------+
|   Локальные       | ← Callee's frame
|   переменные      |
+-------------------+
|   Временные       |
|   данные          |
+-------------------+ ← Низкие адреса
```

### 2. Размер стека в Go

- **Начальный размер**: 2KB (до Go 1.4 было 8KB)
- **Динамическое изменение**: стек может расти и сокращаться
- **Максимальный размер**: зависит от ОС, обычно 1GB на 64-битных системах

```go
// Узнать размер стека
import "runtime/debug"

func main() {
    stackSize := debug.SetMaxStack(64 * 1024 * 1024) // 64MB
    println("Max stack size:", stackSize)
}
```

## Что хранится на стеке

### 1. Локальные переменные

```go
func stackExample() {
    var a int = 42          // На стеке
    var b string = "hello"  // На стеке (заголовок строки)
    var c [100]int          // На стеке (если размер известен)
    
    // Указатели на стеке
    ptr := &a               // Указатель хранится на стеке, указывает на стек
}
```

### 2. Аргументы функций

```go
func add(x, y int) int {  // x и y на стеке
    return x + y
}

func main() {
    result := add(10, 20)  // Аргументы передаются через стек
}
```

### 3. Возвращаемые значения

```go
func calculate() (int, error) {  // Возвращаемые значения на стеке
    return 42, nil
}
```

## Преимущества стека

### 1. Быстрое выделение памяти

```go
func fastAllocation() {
    // Выделение на стеке - одна инструкция (sub ESP, size)
    data := [1000]int{}  // Мгновенное выделение
}
```

### 2. Автоматическое освобождение

```go
func automaticCleanup() {
    resources := acquireResources()  // На стеке
    // Автоматически освобождается при выходе из функции
} // Память освобождается здесь
```

### 3. Кэш-дружественность

```go
func cacheFriendly() {
    // Локальные переменные находятся близко в памяти
    var a, b, c int
    a = 1
    b = 2
    c = a + b  // Все данные в кэше процессора
}
```

## Ограничения стека

### 1. Размер ограничен

```go
func stackOverflow() {
    var huge [1024 * 1024 * 10]byte  // 10MB - может переполнить стек
    // Будет: runtime: goroutine stack exceeds 1000000000-byte limit
}
```

### 2. Время жизни ограничено областью видимости

```go
func cannotReturnStackAddress() *int {
    var x int = 42
    return &x  // ОШИБКА: адрес stack-переменной нельзя вернуть!
}
```

# Куча (Heap) в Go

## Что такое куча

**Куча** - это область динамической памяти, которая доступна всем горутинам. Управляется сборщиком мусора.

## Технические характеристики кучи

### 1. Организация памяти

```
+-------------------+ ← Высокие адреса
|   Large objects   |
+-------------------+
|   Medium spans    |
+-------------------+
|   Small spans     |
+-------------------+
|   Tiny objects    |
+-------------------+
|   Metadata        |
+-------------------+ ← Низкие адреса
```

### 2. Управление памятью

Go использует **mspan** - структуры для управления блоками памяти разных размеров:

- **Tiny**: < 16B
- **Small**: 16B - 32KB  
- **Large**: > 32KB

## Что хранится в куче

### 1. Переменные, которые "убегают"

```go
func escapeToHeap() *int {
    x := 42  // Убегает - в куче
    return &x
}

func main() {
    data := make([]int, 1000)  // В куче (динамический размер)
    _ = data
}
```

### 2. Большие объекты

```go
func largeObjects() {
    // Большие массивы идут в кучу
    var bigArray [1000000]int  // В куче
    _ = bigArray
}
```

### 3. Общие данные

```go
var globalData *Data  // Глобальная переменная - в куче

func sharedData() {
    data := &Data{}  // В куче, так как может быть доступен из других мест
    globalData = data
}
```

## Процесс выделения памяти в куче

### 1. Вызов runtime.mallocgc

```go
// Под капотом make()
slice := make([]int, 100)  // Вызывает runtime.makeslice
// который вызывает runtime.mallocgc
```

### 2. Алгоритм выделения

```go
func allocationProcess() {
    // Малые объекты (<32KB) - из кэша P
    small := make([]byte, 100)
    
    // Большие объекты - напрямую из кучи
    large := make([]byte, 100000)
    
    _ = small
    _ = large
}
```

## Сборка мусора (Garbage Collection)

### 1. Трехцветный алгоритм

```go
func garbageCollectionExample() {
    obj1 := &Object{}  // Белый (не посещен)
    obj2 := &Object{}  // Белый
    
    obj1.ref = obj2    // Создание ссылки
    
    // GC начинает с корней (стек, глобальные переменные)
    // obj1 становится серым, obj2 остается белым
    // obj1 сканируется, obj2 становится серым
    // Все серые объекты становятся черными
    // Белые объекты удаляются
}
```

### 2. Поколения памяти

Go использует **поколенческий GC**:
- **Молодые объекты** - чаще сканируются
- **Старые объекты** - реже сканируются

## Сравнение стека и кучи

### Таблица сравнения

| Характеристика | Стек | Куча |
|----------------|------|------|
| **Скорость выделения** | ⚡️ Очень быстрая | 🐢 Медленная |
| **Освобождение** | Автоматическое | Через GC |
| **Размер** | Ограничен | Большой |
| **Потокобезопасность** | Нет (per-goroutine) | Да |
| **Время жизни** | Область видимости | Пока есть ссылки |
| **Кэш-эффективность** | Высокая | Низкая |

### Производительность

```go
func benchmarkStackVsHeap() {
    // Стек: ~1-2 наносекунды
    var stackVar int
    
    // Куча: ~100-200 наносекунд
    heapVar := new(int)
    
    _ = stackVar
    _ = heapVar
}
```

## Практические примеры

### Пример 1: Контроль размещения

```go
//go:noinline
func createOnStack() Data {
    return Data{Value: 42}  // На стеке (если не убегает)
}

func createOnHeap() *Data {
    return &Data{Value: 42}  // В куче (убегает)
}

func main() {
    stackData := createOnStack()  // На стеке
    heapData := createOnHeap()    // В куче
    
    _ = stackData
    _ = heapData
}
```

### Пример 2: Влияние размера

```go
func sizeMatters() {
    small := [100]int{}    // На стеке (если возможно)
    large := [100000]int{} // В куче (большой размер)
    
    _ = small
    _ = large
}
```

### Пример 3: Замыкания и куча

```go
func closureExample() func() int {
    x := 100  // Убегает в кучу (захватывается замыканием)
    return func() int {
        return x
    }
}
```

## Оптимизации компилятора

### 1. Escape Analysis

```go
func optimized() {
    data := make([]int, 1000)
    // Если data не убегает из функции - может остаться на стеке
    for i := range data {
        data[i] = i
    }
}
```

### 2. Inlining и размещение

```go
func smallFunction() *Data {
    return &Data{Value: 42}  // Кажется, что убегает...
}

func main() {
    data := smallFunction()  // Но после инлайнинга компилятор видит,
    // что data не убегает из main, и может оставить на стеке
}
```

## Мониторинг и диагностика

### 1. Анализ побега переменных

```bash
go build -gcflags="-m" main.go
```

Вывод:
```
./main.go:10:6: can inline createOnHeap
./main.go:11:9: &Data{...} escapes to heap
./main.go:15:6: can inline createOnStack  
./main.go:16:9: Data{...} does not escape
```

### 2. Профилирование памяти

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    // ... ваш код ...
}
```

```bash
# Анализ кучи
go tool pprof http://localhost:6060/debug/pprof/heap
```

### 3. Статистика GC

```go
import (
    "runtime"
    "fmt"
)

func printGCStats() {
    var stats runtime.MemStats
    runtime.ReadMemStats(&stats)
    
    fmt.Printf("Heap alloc: %d\n", stats.HeapAlloc)
    fmt.Printf("Heap sys: %d\n", stats.HeapSys) 
    fmt.Printf("GC cycles: %d\n", stats.NumGC)
}
```

## Best Practices

### Для эффективного использования стека

1. **Локализуйте данные** в функциях
2. **Избегайте ненужных указателей** для маленьких объектов
3. **Используйте значения** для небольших структур

```go
// Хорошо - на стеке
func processUser(user User) error {
    // Работа с копией
    return nil
}

// Плохо - ненужный указатель
func processUser(user *User) error {
    // Указатель для маленькой структуры
    return nil
}
```

### Для эффективного использования кучи

1. **Пулы объектов** для частого создания/удаления
2. **Предварительное выделение** для слайсов и maps
3. **Избегайте утечек памяти**

```go
// Object pool pattern
var userPool = sync.Pool{
    New: func() interface{} { return new(User) },
}

func getUser() *User {
    return userPool.Get().(*User)
}

func putUser(user *User) {
    userPool.Put(user)
}
```

### Оптимизация больших данных

```go
func processLargeData() {
    // Предварительное выделение
    data := make([]int, 0, 1000000)  // Один allocation
    
    for i := 0; i < 1000000; i++ {
        data = append(data, i)  // Без переаллокаций
    }
}
```

## Особые случаи

### 1. Интерфейсы и куча

```go
func interfaceExample() {
    var val interface{} = 42  // int убегает в кучу (boxing)
    
    // Методы с приемниками по значению
    var w io.Writer = &Buffer{}  // Buffer может остаться на стеке
}
```

### 2. Рефлексия и куча

```go
func reflectionExample() {
    var x int = 42
    value := reflect.ValueOf(x)  // x может убежать в кучу
    
    _ = value
}
```

### 3. CGo и память

```go
/*
#include <stdlib.h>
*/
import "C"

func cgoExample() {
    // Память, выделенная в C, управляется вручную
    cstr := C.CString("hello")
    defer C.free(unsafe.Pointer(cstr))
}
```

Понимание различий между стеком и кучей критически важно для написания эффективных программ на Go. Правильное использование этих областей памяти позволяет значительно улучшить производительность и снизить нагрузку на сборщик мусора.