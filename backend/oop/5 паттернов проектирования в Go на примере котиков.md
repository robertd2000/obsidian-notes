https://habr.com/ru/companies/otus/articles/864748/

Привет, Хабр! Сегодня мы рассмотрим реализацию паттернов проектирования на Go, и, чтобы было не скучно, возьмем главными героями котиков. Будем разбирать 5 популярных паттернов: **Singleton**, **Factory Method**, **Strategy**, **Observer**, **Decorator**.

### Singleton

Паттерн будет хорош тогда, когда в компании есть один кот, который отвечает, например, за доступ к единственной миске с едой, мы хотим быть уверены, что он существует в единственном экземпляре, и все остальные обращаются к нему как к глобальному ресурсу. Допустим, есть global-cat, который знает, где самая вкусная еда, и поэтому мы делаем Singleton:

```go
package main

import (
	"fmt"
	"sync"
)

// SupremeCat — структура для Singleton
type SupremeCat struct {
	Name string
}

var (
	once       sync.Once
	supremeCat *SupremeCat
)

// GetSupremeCat — возвращает единственный экземпляр SupremeCat
func GetSupremeCat() *SupremeCat {
	once.Do(func() {
		fmt.Println("Создаётся экземпляр SupremeCat...")
		supremeCat = &SupremeCat{Name: "Барсик Великий"}
	})
	return supremeCat
}

func main() {
	cat1 := GetSupremeCat()
	cat2 := GetSupremeCat()
	
	if cat1 == cat2 {
		fmt.Println("Мы имеем один и тот же единственный кот:", cat1.Name)
	} else {
		fmt.Println("Что-то пошло не так, у нас больше одного кота!")
	}
}
```

`sync.Once` гарантирует, что инициализация кота пройдет ровно один раз даже в многопоточной среде — никакой гонки.

Singleton нужен там, где необходимо гарантировать существование ровно одного экземпляра объекта. Это может быть:

- **Connection pool к базе данных**. Например, если приложение работает с одной базой, и мы хотим избежать создания лишних соединений.
    
- **Логгер**. Зачем плодить несколько логгеров, если можно централизовать запись логов?
    
- **Кеши или конфигурации**. Общий доступ к конфигурации приложения — классический случай.
    

### Factory Method

Представим, что есть разные типы котов: домашний кот (ленивый, может лежать на диване и мурлыкать) и уличный кот (хулиганистый, любит охотиться за голубями). Мы хотим иметь удобный фабричный метод, который, по какому-то параметру, будет создавать нужный тип кота, не раскрывая деталей его конструктора.

```go
package main

import (
	"errors"
	"fmt"
)

// Cat — интерфейс для котов
type Cat interface {
	Meow()
	GetLifestyle() string
}

// HomeCat — домашний кот
type HomeCat struct {
	name string
}

func (h *HomeCat) Meow() {
	fmt.Println(h.name, "спокойно мурлычет на диване.")
}

func (h *HomeCat) GetLifestyle() string {
	return "Домашний кот, любит кушать и спать."
}

// StreetCat — уличный кот
type StreetCat struct {
	name string
}

func (s *StreetCat) Meow() {
	fmt.Println(s.name, "громко орёт во дворе и ворует сосиски.")
}

func (s *StreetCat) GetLifestyle() string {
	return "Уличный кот, охотник и бандит."
}

// NewCat — фабричный метод для создания котов
func NewCat(catType, name string) (Cat, error) {
	if name == "" {
		return nil, errors.New("имя кота не может быть пустым")
	}
	
	switch catType {
	case "home":
		return &HomeCat{name: name}, nil
	case "street":
		return &StreetCat{name: name}, nil
	default:
		return nil, fmt.Errorf("неизвестный тип кота: %s", catType)
	}
}

func main() {
	// Создаём домашнего кота
	cat1, err := NewCat("home", "Пушок")
	if err != nil {
		fmt.Println("Ошибка создания кота:", err)
		return
	}
	cat1.Meow()
	fmt.Println(cat1.GetLifestyle())
	
	// Создаём уличного кота
	cat2, err := NewCat("street", "Рыжик")
	if err != nil {
		fmt.Println("Ошибка создания кота:", err)
		return
	}
	cat2.Meow()
	fmt.Println(cat2.GetLifestyle())
	
	// Пробуем создать кота с неизвестным типом
	cat3, err := NewCat("wild", "Дикий")
	if err != nil {
		fmt.Println("Ошибка создания кота:", err)
		return
	}
	cat3.Meow()
	fmt.Println(cat3.GetLifestyle())
}
```

Мы скрываем конкретные реализации за фабричным методом `NewCat`. Можно быстро подкинуть новый тип кота, не меняя код клиентов — просто дополнить фабрику.

Фабричный метод идеален, когда:

- Есть семейство объектов с разными реализациями, но клиентский код не должен зависеть от деталей их создания.
    
- Нужно централизовать и стандартизировать логику создания объектов.
    

### Strategy

Представим, что есть кот, который в зависимости от ситуации мяукает по-разному. Иногда он «мяучит просяще», когда хочет поесть. Иногда — грозно, когда защищает территорию. Иногда тихо, когда пытается пробраться в комнату незаметно. Хочется подменять поведение «мяу» на лету, не меняя класс кота.

```go
package main

import "fmt"

// MeowStrategy — интерфейс для стратегий мяуканья
type MeowStrategy interface {
	Meow(name string)
}

// QuietMeow — тихое мяуканье
type QuietMeow struct{}

func (q *QuietMeow) Meow(name string) {
	fmt.Println(name, "тихо и незаметно подмяукивает...")
}

// BeggingMeow — жалобное мяуканье
type BeggingMeow struct{}

func (b *BeggingMeow) Meow(name string) {
	fmt.Println(name, "жалобно просит еду: \"Мяу! Мяу!\"")
}

// AngryMeow — агрессивное мяуканье
type AngryMeow struct{}

func (a *AngryMeow) Meow(name string) {
	fmt.Println(name, "яростно шипит и орёт: \"МЯУ!\"")
}

// FlexibleCat — кот с изменяемым поведением мяуканья
type FlexibleCat struct {
	name        string
	meowBehavior MeowStrategy
}

// PerformMeow вызывает текущее поведение мяуканья
func (f *FlexibleCat) PerformMeow() {
	if f.meowBehavior == nil {
		fmt.Println(f.name, "не знает, как мяукать!")
		return
	}
	f.meowBehavior.Meow(f.name)
}

// SetMeowStrategy устанавливает новую стратегию мяуканья
func (f *FlexibleCat) SetMeowStrategy(strategy MeowStrategy) {
	if strategy == nil {
		fmt.Println("Невозможно установить пустую стратегию для", f.name)
		return
	}
	f.meowBehavior = strategy
	fmt.Println(f.name, "сменил стиль мяуканья!")
}

func main() {
	// Создаём кота с тихим мяуканьем
	cat := &FlexibleCat{name: "Тишка", meowBehavior: &QuietMeow{}}
	cat.PerformMeow()
	
	// Меняем поведение на жалобное
	cat.SetMeowStrategy(&BeggingMeow{})
	cat.PerformMeow()
	
	// Меняем поведение на агрессивное
	cat.SetMeowStrategy(&AngryMeow{})
	cat.PerformMeow()
	
	// Пробуем установить nil стратегию
	cat.SetMeowStrategy(nil)
	cat.PerformMeow()
}
```

Strategy полезен, когда:

- У объекта есть несколько вариантов поведения, и нужно переключаться между ними без переписывания класса.
    
- Нужно инкапсулировать поведение и сделать код более тестируемым.
    

### Observer

Представим, что есть кот, который совершает некие действия (например, ест, спит, играет), и есть другие компоненты системы, которым надо знать, что кот сделал что-то интересное. Например, есть мониторинг, который отслеживает, сколько раз кот кушал, чтобы не дать ему переесть. Есть логгер, который записывает все действия кота. Observer паттерн позволит подписаться на события кота и автоматически оповещать подписчиков.

```go
package main

import (
	"fmt"
	"strings"
)

// Observer — интерфейс для наблюдателей
type Observer interface {
	Notify(action string)
}

// CatSubject — кот, на чьи действия подписываются наблюдатели
type CatSubject struct {
	observers []Observer
	name      string
}

// Attach добавляет наблюдателя
func (c *CatSubject) Attach(o Observer) {
	if o == nil {
		return
	}
	c.observers = append(c.observers, o)
}

// Detach удаляет наблюдателя
func (c *CatSubject) Detach(o Observer) {
	newObservers := make([]Observer, 0, len(c.observers))
	for _, obs := range c.observers {
		if obs != o {
			newObservers = append(newObservers, obs)
		}
	}
	c.observers = newObservers
}

// NotifyAll оповещает всех наблюдателей
func (c *CatSubject) NotifyAll(action string) {
	for _, obs := range c.observers {
		if obs != nil {
			obs.Notify(fmt.Sprintf("%s: %s", c.name, action))
		}
	}
}

// LogObserver пишет в лог действия кота
type LogObserver struct{}

func (l *LogObserver) Notify(action string) {
	fmt.Println("[LOG]:", action)
}

// HealthObserver следит за количеством еды
type HealthObserver struct {
	eatCount int
}

func (h *HealthObserver) Notify(action string) {
	if strings.Contains(action, "ест") {
		h.eatCount++
		if h.eatCount > 3 {
			fmt.Println("[HEALTH]: Кот переел! Нужно ограничить доступ к миске!")
		} else {
			fmt.Printf("[HEALTH]: Кот покушал. Всего раз: %d\n", h.eatCount)
		}
	} else {
		fmt.Println("[HEALTH]: Кот делает что-то не связанное с едой.")
	}
}

// Действия кота
func (c *CatSubject) Eat() {
	c.NotifyAll("ест корм")
}

func (c *CatSubject) Sleep() {
	c.NotifyAll("сладко спит")
}

func (c *CatSubject) Play() {
	c.NotifyAll("играет с мячиком")
}

func main() {
	cat := &CatSubject{name: "Мурзик"}
	logObs := &LogObserver{}
	healthObs := &HealthObserver{}
	
	// Подписываем наблюдателей
	cat.Attach(logObs)
	cat.Attach(healthObs)
	
	// Действия кота
	cat.Eat()
	cat.Eat()
	cat.Play()
	cat.Eat()
	cat.Eat()
	cat.Sleep()
	
	// Отписываем логгера
	cat.Detach(logObs)
	
	// Действия после отписки
	cat.Eat()
}
```

Логи пишутся, здоровье мониторится.

Примеры использования:

- **Уведомления в UI**.
    
- **Логирование событий**.
    

### Decorator

Декоратор позволяет обернуть кота в дополнительные способности, не меняя исходный код кота. Представим, что есть базовый кот, который умеет просто мяукать, а нам нужно декорировать его возможностями: например, кот после мяуканья может автоматически отправлять событие в Slack или писать в лог. Добавим декоратор, который добавляет поведение сверху.

```go
package main

import "fmt"

// SimpleCat — интерфейс кота
type SimpleCat interface {
	Meow()
}

// BaseCat — базовый кот
type BaseCat struct {
	name string
}

func (b *BaseCat) Meow() {
	fmt.Println(b.name, "просто говорит: 'Мяу!'")
}

// LoggingCatDecorator — декоратор, логирующий мяуканье
type LoggingCatDecorator struct {
	cat SimpleCat
}

func (l *LoggingCatDecorator) Meow() {
	if l.cat == nil {
		fmt.Println("[LOGGING ERROR]: Нет кота для логирования!")
		return
	}
	fmt.Println("[LOGGING BEFORE]: Кот готовится к мяу.")
	l.cat.Meow()
	fmt.Println("[LOGGING AFTER]: Кот уже мяукнул.")
}

// NotifyingCatDecorator — декоратор, отправляющий уведомления
type NotifyingCatDecorator struct {
	cat SimpleCat
}

func (n *NotifyingCatDecorator) Meow() {
	if n.cat == nil {
		fmt.Println("[NOTIFY ERROR]: Нет кота для уведомления!")
		return
	}
	fmt.Println("[NOTIFY]: Отправляем сообщение в Slack канал #cats")
	n.cat.Meow()
}

func main() {
	// Создаём базового кота
	cat := &BaseCat{name: "Котофей"}
	
	// Добавляем декоратор логирования
	loggedCat := &LoggingCatDecorator{cat: cat}
	
	// Добавляем декоратор уведомлений
	notifiedAndLoggedCat := &NotifyingCatDecorator{cat: loggedCat}
	
	// Обычный кот
	fmt.Println("=== Обычный кот ===")
	cat.Meow()
	
	// Кот с логированием
	fmt.Println("\n=== Кот с логированием ===")
	loggedCat.Meow()
	
	// Кот с логированием и уведомлением
	fmt.Println("\n=== Кот с логированием и уведомлением ===")
	notifiedAndLoggedCat.Meow()
}
```

В продакшне декоратор — классический способ расширить функционал без ломки существующего кода. Можно цеплять дополнительные фичи к чему угодно: расширять логирование, добавлять трейсинг, кэширование, всё что угодно. Тут кот просто мяукает, а мы сверху навешиваем логгер и отправку сообщения.

---

#### Заключение

Как видите, не важно, что у нас за доменная область: коты, собаки, сервисы аутентификации или раздатчики пиццы — паттерны универсальны. Главное, понять, где применить их грамотно, и помнить, что паттерн — это не панацея, а инструмент.

Хочу порекомендовать вам бесплатные вебинары курса "Архитектура и шаблоны проектирования":

- 11.12. Создание микросервиса. [Зарегистрироваться](https://otus.pw/V76O/)
    
- 18.12. Обработка исключений и SOLID. [Зарегистрироваться](https://otus.pw/Syqi/)
- 