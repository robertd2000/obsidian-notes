https://www.youtube.com/watch?v=KJIchjXmXJ8

# 1 interface

https://www.youtube.com/watch?v=7_h9iT672HQ


# 2 чем платим за использование интерфейсов 

2 указателя - (16 byte)
+доп память

# 3 индирекция

ИНдирекция это когда приходится перемещаться по нескольким указателям чтобы дойти до данных

В Go (Golang) **индирекция** (indirection) обычно относится к использованию указателей и интерфейсов для работы с данными косвенно. Вот основные аспекты:

### 1. Указатели (Pointers) - базовая индирекция

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    // Прямой доступ
    p := Person{Name: "Alice", Age: 30}
    fmt.Println(p.Name) // Прямой доступ
    
    // Индирекция через указатель
    ptr := &p
    fmt.Println(ptr.Name)   // Автоматическое разыменование
    fmt.Println((*ptr).Name) // Явное разыменование
    
    // Модификация через указатель
    modifyPerson(ptr)
    fmt.Println(p.Name) // "Bob"
}

func modifyPerson(p *Person) {
    p.Name = "Bob" // Изменение оригинала
}
```

### 2. Интерфейсы (Interfaces) - абстрактная индирекция

```go
package main

import "fmt"

type Writer interface {
    Write([]byte) (int, error)
}

type FileWriter struct{}

func (fw FileWriter) Write(data []byte) (int, error) {
    fmt.Println("Writing to file:", string(data))
    return len(data), nil
}

type NetworkWriter struct{}

func (nw NetworkWriter) Write(data []byte) (int, error) {
    fmt.Println("Sending over network:", string(data))
    return len(data), nil
}

func processData(w Writer, data string) {
    w.Write([]byte(data)) // Индирекция через интерфейс
}

func main() {
    fileWriter := FileWriter{}
    networkWriter := NetworkWriter{}
    
    processData(fileWriter, "file data")    // Writing to file
    processData(networkWriter, "net data")  // Sending over network
}
```

### 3. reflect.Value - индирекция во время выполнения

```go
package main

import (
    "fmt"
    "reflect"
)

type Config struct {
    Host string
    Port int
}

func setFieldIndirectly(obj interface{}, fieldName string, value interface{}) {
    v := reflect.ValueOf(obj).Elem()
    field := v.FieldByName(fieldName)
    
    if field.IsValid() && field.CanSet() {
        field.Set(reflect.ValueOf(value))
    }
}

func main() {
    config := Config{Host: "localhost", Port: 8080}
    fmt.Println("Before:", config)
    
    setFieldIndirectly(&config, "Port", 9090)
    fmt.Println("After:", config) // Port: 9090
}
```

### 4. unsafe.Pointer - низкоуровневая индирекция

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    var x int64 = 42
    var y float64
    
    // Конвертация через unsafe индирекцию
    ptr := unsafe.Pointer(&x)
    floatPtr := (*float64)(ptr)
    y = *floatPtr
    
    fmt.Printf("int64: %d, float64: %f\n", x, y)
}
```

### 5. Индирекция в каналах

```go
package main

import "fmt"

func processThroughChannel(data *string, ch chan<- *string) {
    *data = "processed: " + *data
    ch <- data // Передача указателя через канал
}

func main() {
    data := "original"
    ch := make(chan *string)
    
    go processThroughChannel(&data, ch)
    
    result := <-ch
    fmt.Println(*result) // "processed: original"
    fmt.Println(data)    // "processed: original" (тот же объект)
}
```

### Практические применения индирекции:

1. **Эффективная передача больших структур**
2. **Изменение переданных параметров**
3. **Реализация полиморфизма через интерфейсы**
4. **Работа с данными во время выполнения (reflection)**
5. **Оптимизация производительности**

### Важные особенности Go:

- Автоматическое разыменование указателей при доступе к полям структур
- Интерфейсы неявно реализуются
- `nil` указатели и интерфейсы - разные понятия
- Сборщик мусора корректно работает с указателями

Индирекция в Go предоставляет мощные инструменты для создания гибких и эффективных программ while maintaining type safety.
# 4 Редирекция

# 5 Переход по указателю - бесплатная операция?
# 6 defer в теле цикла 

LIFO
# 7 наследование
# 8 композиция
# 9 почему композиция лучше наследования

Композиция предпочтительнее наследования по нескольким ключевым причинам:

## 1. Гибкость и избегание хрупкости

**Наследование** создает жесткую связь:
```go
// Проблема наследования
type Bird struct {
    Name string
}

func (b *Bird) Fly() {
    fmt.Println("Flying")
}

type Penguin struct {
    Bird // Наследование
}

// Пингвин не может летать, но наследует метод Fly!
penguin := Penguin{}
penguin.Fly() // ❌ Нелогично
```

**Композиция** решает проблему:
```go
type Flyer interface {
    Fly()
}

type Wing struct {
    Size string
}

func (w *Wing) Fly() {
    fmt.Println("Flying with wings")
}

type Penguin struct {
    Name  string
    Wings *Wing // Композиция
}

// Пингвин использует крылья только для плавания
func (p *Penguin) Swim() {
    fmt.Println("Swimming")
}
```

## 2. Избегание проблем diamond problem

**Проблема наследования** (в языках с множественным наследованием):
```
    A
   / \
  B   C
   \ /
    D
```

**Композиция решает**:
```go
type A struct {
    Value string
}

type B struct {
    AComponent A // Композиция вместо наследования
}

type C struct {
    AComponent A // Композиция вместо наследования
}

type D struct {
    BComponent B
    CComponent C
}

func (d *D) GetValue() string {
    return d.BComponent.AComponent.Value
}
```

## 3. Принцип единственной ответственности

**Наследование нарушает SRP**:
```go
type Vehicle struct {
    EngineType string
    WheelCount int
}

// Car наследует ВСЕ методы Vehicle, даже если они не нужны
type Car struct {
    Vehicle
    Brand string
}
```

**Композиция соблюдает SRP**:
```go
type Engine struct {
    Type string
}

type Wheels struct {
    Count int
}

type Car struct {
    Engine  Engine  // Только то, что нужно
    Wheels  Wheels  // Только то, что нужно
    Brand   string
}
```

## 4. Легкость тестирования

**Наследование сложно тестировать**:
```go
type Database struct {
    Connection string
}

func (d *Database) Query() string {
    // Сложная логика базы данных
    return "data"
}

type Service struct {
    Database // Наследование
}

// Тестирование Service требует мокирования Database
```

**Композиция упрощает тестирование**:
```go
type Database interface {
    Query() string
}

type RealDatabase struct {
    Connection string
}

func (d *RealDatabase) Query() string {
    return "real data"
}

type Service struct {
    DB Database // Композиция через интерфейс
}

// Легко подменить реализацию для тестов
type MockDatabase struct{}

func (m *MockDatabase) Query() string {
    return "mock data"
}

func TestService() {
    service := Service{DB: &MockDatabase{}}
    // Простое тестирование
}
```

## 5. Эволюция API без breaking changes

**Наследование ломает клиентов**:
```go
type Base struct {
    Value int
}

func (b *Base) Process() {
    // Базовая реализация
}

type Derived struct {
    Base
}

// Если изменить Base.Process(), все Derived ломаются
```

**Композиция защищает от изменений**:
```go
type Processor interface {
    Process()
}

type DefaultProcessor struct{}

func (d *DefaultProcessor) Process() {
    // Базовая реализация
}

type Client struct {
    Processor Processor // Зависимость от интерфейса
}

// Можно менять реализации, не затрагивая клиентов
```

## 6. Переиспользование кода без связывания

**Наследование создает сильную связь**:
```go
type ArrayList struct {
    elements []interface{}
}

// LinkedList наследует ненужные методы ArrayList
type LinkedList struct {
    ArrayList // ❌ Плохая идея
}
```

**Композиция позволяет выбирать поведение**:
```go
type List interface {
    Add(item interface{})
    Get(index int) interface{}
}

type ArrayList struct {
    elements []interface{}
}

type LinkedList struct {
    head *Node
}

type Collection struct {
    List List // Используем нужную реализацию
}

// Можно легко менять реализацию списка
collection := Collection{List: &ArrayList{}}
collection.List = &LinkedList{} // Без проблем
```

## 7. Реализация в Go

Go намеренно не имеет классического наследования, продвигая композицию:

```go
// Правильный подход в Go
type Reader interface {
    Read() []byte
}

type Writer interface {
    Write([]byte) error
}

// Композиция интерфейсов
type ReadWriter interface {
    Reader
    Writer
}

// Композиция структур
type FileSystem struct {
    Reader Reader
    Writer Writer
}

func (fs *FileSystem) Copy() error {
    data := fs.Reader.Read()
    return fs.Writer.Write(data)
}
```

## 8. Соответствие принципам SOLID

**Композиция лучше соответствует**:
- **Open/Closed** - Легко расширять через новые компоненты
- **Liskov Substitution** - Нет риска нарушения контракта
- **Interface Segregation** - Используем только нужные интерфейсы
- **Dependency Inversion** - Зависим от абстракций

## Практический пример: система оплаты

**Плохо через наследование**:
```go
type PaymentProcessor struct {
    APIKey string
}

func (p *PaymentProcessor) Process(amount float64) {
    // Общая логика
}

type PayPalProcessor struct {
    PaymentProcessor // Наследование
    // ❌ Привязываемся к реализации родителя
}
```

**Хорошо через композицию**:
```go
type PaymentAuthorizer interface {
    Authorize() bool
}

type PaymentLogger interface {
    LogTransaction()
}

type PayPalProcessor struct {
    Authorizer PaymentAuthorizer
    Logger     PaymentLogger
}

func (p *PayPalProcessor) Process(amount float64) {
    if p.Authorizer.Authorize() {
        p.Logger.LogTransaction()
    }
}
```

## Вывод

**Используйте композицию когда**:
- Нужна гибкость и переиспользование
- Хотите избежать жестких связей
- Важен простой тестинг
- Планируете эволюцию системы

**Наследование уместно когда**:
- Есть строгая иерархия "is-a"
- Поведение действительно должно наследоваться
- Язык предоставляет безопасное наследование (как в Go через embedding)

В Go **композиция + интерфейсы** - это идеальный способ построения гибких и поддерживаемых систем.
# 10 bytes buffer - зачем буфферизация
# 11 примитивы синхронизации
# 12 как остановить горутину извне
# 13 ограничения отсановки горутин
# 14 как принудительно остановить горутину
нельзя
# 15 сколько init функций может быть в одном пакете
одна
# 16 как устроена map под капотом
# 17 Эвакуация
# 18 select
# 19 
# 20
# 21
# 22
# 23
# 24
# 25
# 26






Таймкоды: [00:00](https://www.youtube.com/watch?v=KJIchjXmXJ8) – Введение [00:18](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=18s) – Введение в компанию Уверенность: Высокая. Правильность: Корректно описал направления компании, системы WMS и использование ТСД. [01:50](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=110s) – Жизненный цикл товара Уверенность: Высокая. Правильность: Четко описал этапы жизненного цикла товара на складе. [03:16](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=196s) – Новые направления и команды Уверенность: Средняя. Правильность: Описал новые проекты и состав команд, но недостаточно детализировал роли. [05:23](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=323s) – Методология работы Уверенность: Высокая. Правильность: Корректно описал процесс формирования задач и используемые технологии. [10:12](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=612s) – Оценка и планы Уверенность: Средняя. Правильность: Оценил себя на 7/10, указал слабые стороны в архитектуре и углублении в Go. [12:06](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=726s) – Теоретические вопросы (интерфейсы) Уверенность: Средняя. Правильность: Корректно описал затраты памяти на интерфейсы (16 байт), но не упомянул дополнительные накладные расходы. [14:21](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=861s) – Редирекция и индирекция Уверенность: Низкая. Правильность: Неправильно определил индирекцию, но верно отметил, что переход по указателю не бесплатный. [15:17](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=917s) – Выполнение defer в цикле Уверенность: Средняя. Правильность: Ошибся с порядком выполнения (LIFO, а не FIFO), но верно указал момент выполнения defer. [16:25](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=985s) – Наследование и композиция Уверенность: Средняя. Правильность: Корректно отметил отсутствие наследования в Go и преимущества композиции. [17:51](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=1071s) – Буферизация и примитивы синхронизации Уверенность: Высокая. Правильность: Корректно описал цель буферизации и перечислил примитивы синхронизации. [19:18](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=1158s) – Остановка горутины извне Уверенность: Средняя. Правильность: Верно указал использование контекста и каналов, но не упомянул ограничения остановки горутин. [21:20](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=1280s) – Программирование задачи (настройка среды) Уверенность: Низкая. Правильность: Возникли технические проблемы с настройкой среды разработки. [28:15](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=1695s) – Реализация проверки доступности Уверенность: Средняя. Правильность: Корректно начал реализацию проверки хостов через карту, но не учел обработку ошибок. [30:39](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=1839s) – Обработка ошибок и завершение проверки Уверенность: Средняя. Правильность: Верно предложил возвращать первую ошибку, но не сразу учел отмену всех проверок. [37:36](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=2256s) – Обсуждение ошибок и параллельного запуска Уверенность: Средняя. Правильность: Корректно предложил параллельный запуск в горутинах для оптимизации. [39:49](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=2389s) – Создание и использование wait-групп Уверенность: Средняя. Правильность: Верно использовал wait-группу, но ошибся с указателями, что было исправлено. [44:49](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=2689s) – Архитектура и работа с картами Уверенность: Средняя. Правильность: Корректно описал работу мапы (хэш-функция, ведра), но упростил эвакуацию данных. [46:46](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=2806s) – Проблемы с добавлением хостов и их решение Уверенность: Средняя. Правильность: Верно определил проблему с добавлением хостов, предложил копирование мапы и мьютексы. [50:41](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3041s) – Использование контекста для обработки ошибок Уверенность: Средняя. Правильность: Корректно предложил контекст для остановки проверок при ошибке. [56:21](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3381s) – Проблемы с блокирующими вызовами Уверенность: Средняя. Правильность: Верно определил проблему блокирующих вызовов, предложил неблокирующие запросы. [59:50](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3590s) – Проблемы с контекстом и тайм-аутами Уверенность: Средняя. Правильность: Корректно предложил контекст с тайм-аутом для защиты от медленных хостов. [01:00:44](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3644s) – Оптимизация кода Уверенность: Высокая. Правильность: Верно предложил сократить условные операторы для оптимизации. [01:02:50](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3770s) – Вопросы по организации работы Уверенность: Высокая. Правильность: Корректно описал состав команды, архитектурный подход и задачи. [01:04:18](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3858s) – Условия работы и оборудование Уверенность: Высокая. Правильность: Точно описал условия удаленной работы, оборудование и график. [01:05:49](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=3949s) – Вертикальный рост и перемещение между командами Уверенность: Высокая. Правильность: Корректно описал возможности роста и перемещения между командами. [01:07:03](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=4023s) – Методология разработки и фичи Уверенность: Высокая. Правильность: Четко описал Kanban, декомпозицию задач и примеры фич. [01:09:04](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=4144s) – Последние задачи и эпики Уверенность: Высокая. Правильность: Подробно описал эпики и декомпозицию задач для модуля сканирования. [01:11:02](https://www.youtube.com/watch?v=KJIchjXmXJ8&t=4262s) – Завершение встречи