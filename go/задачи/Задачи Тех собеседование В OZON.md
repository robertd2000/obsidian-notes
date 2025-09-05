

https://www.youtube.com/watch?v=-8JlOr3Z0eA

# 1

```go

package main

import (
	"fmt"
	"net/http"
)

// Необходимо поочередно выполнить HTTP-запросы по предложенному списку ссылок:

// - Если получен HTTP-код ответа "200 OK" - вывести "адрес url - ok"

// - Если получен любой другой HTTP-код ответа или произошла ошибка - вывести "адрес url - not ok"

func main() {
	var urls = []string{
		"http://ozon.ru",
		"https://ozon.ru",
		"http://google.com",
		"http://somelite.com",
		"http://non-existent.domain.it",
		"https://ya.ru",
		"http://ya.ru",
		"http://eee",
	}

	for _, url := range urls {
		resp, err := http.Get(url)

		if err != nil {
			fmt.Println("err", url)
			continue
		}
		defer resp.Body.Close()

		if resp.StatusCode != http.StatusOK {
			fmt.Println("not ok")
		} else {
			fmt.Println("ok")
		}
	}
}
```

# 2

```go
package main

import (
	"fmt"
	"net/http"
	"sync"
)

// Необходимо поочередно выполнить HTTP-запросы по предложенному списку ссылок:

// - Если получен HTTP-код ответа "200 OK" - вывести "адрес url - ok"

// - Если получен любой другой HTTP-код ответа или произошла ошибка - вывести "адрес url - not ok"
// 2. Модифицируйте программу таким образом, чтобы использовать каналы для коммуникаций основного потока с горутинами. Пример:
// Запросы по списку выполняются в горутинах.
// Печать результатов на экран происходит в основном потоке

func main() {
	var urls = []string{
		"http://ozon.ru",
		"https://ozon.ru",
		"http://google.com",
		"http://somelite.com",
		"http://non-existent.domain.it",
		"https://ya.ru",
		"http://ya.ru",
		"http://eee",
	}
	n := len(urls)
	ch := make(chan string, n)
	wg := &sync.WaitGroup{}

	wg.Add(n)

	for _, url := range urls {
		go func(u string) {
			defer wg.Done()
			resp, err := http.Get(u)

			if err != nil {
				ch <- fmt.Sprintf("адрес %s - not ok", u)
				return
			}
			defer resp.Body.Close()

			if resp.StatusCode != http.StatusOK {
				ch <- fmt.Sprintf("адрес %s - not ok", u)
			} else {
				ch <- fmt.Sprintf("адрес %s - ok", u)
			}

		}(url)
	}

	go func() {
		wg.Wait()
		close(ch)
	}()

	for v := range ch {
		fmt.Println(v)
	}
}

```

# 3

```go

package main

import (
	"fmt"
	"net/http"
	"sync"
	"context"
	"time"
)

// Необходимо поочередно выполнить HTTP-запросы по предложенному списку ссылок:

// - Если получен HTTP-код ответа "200 OK" - вывести "адрес url - ok"

// - Если получен любой другой HTTP-код ответа или произошла ошибка - вывести "адрес url - not ok"
// 2. Модифицируйте программу таким образом, чтобы использовать каналы для коммуникаций основного потока с горутинами. Пример:
// Запросы по списку выполняются в горутинах.
// Печать результатов на экран происходит в основном потоке
//  3 Урлы приходят из внешнего источника, длина неизвестна

const parrallelCount = 10

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	
	urlsChan := generateUrls(ctx)
	res := make(chan string)
	wg := &sync.WaitGroup{}


	for i := 0; i < parrallelCount; i++ {
		wg.Add(1)
		go worker(ctx, urlsChan, res, wg)
	}

	go func() {
		wg.Wait()
		close(res)
	}()

	for v := range res {
		fmt.Println(v)
	}
}

func worker(ctx context.Context, out <- chan string, in chan <- string, wg *sync.WaitGroup) {
	defer wg.Done()

	select {
		case <-ctx.Done():
			return
		case url, ok := <- out:
			if !ok {
				return
			}
			processURL(ctx, url, in)
	}
}

func processURL(ctx context.Context, url string, results chan<- string) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        select {
        case <-ctx.Done():
            return
        default:
            results <- fmt.Sprintf("адрес %s - not ok (ошибка создания запроса)", url)
        }
        return
    }
    
    client := &http.Client{
        Timeout: 5 * time.Second,
    }
    
    resp, err := client.Do(req)
    if err != nil {
        select {
        case <-ctx.Done():
            return 
        default:
            results <- fmt.Sprintf("адрес %s - not ok (%v)", url, err)
        }
        return
    }
    defer resp.Body.Close()
    
    select {
    case <-ctx.Done():
        return 
    default:
        if resp.StatusCode == http.StatusOK {
            results <- fmt.Sprintf("адрес %s - ok", url)
        } else {
            results <- fmt.Sprintf("адрес %s - not ok (status: %d)", url, resp.StatusCode)
        }
    }
}

func generateUrls(ctx context.Context) <-chan string {
	urls := []string{
		"http://ozon.ru",
		"https://ozon.ru",
		"http://google.com",
		"http://somelite.com",
		"http://non-existent.domain.it",
		"https://ya.ru",
		"http://ya.ru",
		"http://eee",
	}
	
	urlChan := make(chan string)
	
	go func() {
		defer close(urlChan)
		
		for _, url := range urls {
			select {
			case <-ctx.Done():
				return
			case urlChan <- url:
			}
			
		}
	}()
	
	return urlChan
}

```

# 2

```go

package main

import (
    "database/sql"
    "fmt"
    "io"
    "net/http"
    "os"
    "strconv"
    "strings"

    _ "github.com/lib/pq"
)

// T3: мы хотим собирать информацию по курсам валют в разных банках.
// Требуется написать программу, которая будет делать запросы в банки и получать курс нескольких валют, и сохранять результат в БД.
// Банков может быть несколько.

func main() {
    if len(os.Args) == 2 {
        cmdName := os.Args[1]
        if cmdName == "help" {
            fmt.Println("Usage is './currency update'")
        } else if cmdName == "update" {
            urlsBank := []struct {
                bankName string
                curFrom  string
                curTo    string
                url      string
            }{
                {
                    bankName: "Bank_1",
                    curFrom:  "RUB",
                    curTo:    "USD",
                    url:      "https://bank.example.com/rates/rub-usd",
                },
                {
                    bankName: "Bank_2",
                    curFrom:  "RUB",
                    curTo:    "USD",
                    url:      "https://bank2.example.com/rates?curFrom=RUB&curTo=USD",
                },
            }

            clientBank := &http.Client{}
            for _, url := range urlsBank {
                req, _ := http.NewRequest(http.MethodGet, url.url, nil)
                if url.bankName == "Bank_2" {
                    req.Header.Add("Authorization", "auth_token=\"XXXXXXXX\"")
                }

                resp, err := clientBank.Do(req)
                if err != nil {
                    panic(err)
                }
                defer resp.Body.Close()
                body, _ := io.ReadAll(resp.Body)

                strBody := string(body)

                if url.bankName == "Bank_1" {
                    strBody = strings.ReplaceAll(strBody, ",", ".") // Заменяем для Банка 1 запятую на точку
                }

                value, err := strconv.ParseFloat(strBody, 64)
                if err != nil {
                    panic(err)
                }
                err = updateCurrency(url.bankName, url.curFrom, url.curTo, value)
                if err != nil {
                    panic(err)
                }
            }
        }
    } else {
        fmt.Println("Usage is './currency update'")
    }
}

const (
    host     = "localhost"
    port     = 5432
    user     = "postgres"
    password = "<password>"
    dbname   = "<dbname>"
)

func updateCurrency(bank, from, to string, value float64) error {
    psqlconn := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", host, port, user, password, dbname)

    db, err := sql.Open("postgres", psqlconn)
    checkError(err)

    defer db.Close()

    err = db.Ping()
    checkError(err)

    fmt.Println("Connected!")

    insertStmt := fmt.Sprintf("insert into currency_rates (bank, from, to, value) values('%s', '%s', '%s', '%.2f')", bank, from, to, value)
    _, err = db.Exec(insertStmt)
    return err
}

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

```

# Код ревью

## Критические проблемы

### 1. **SQL injection уязвимость**
```go
// ОПАСНО: прямое вставление значений в SQL запрос
insertStmt := fmt.Sprintf("insert into currency_rates (bank, from, to, value) values('%s', '%s', '%s', '%.2f')", bank, from, to, value)
```

**Исправление:**
```go
// Использовать подготовленные выражения
insertStmt := `INSERT INTO currency_rates (bank, from_currency, to_currency, value) 
               VALUES ($1, $2, $3, $4)`
_, err = db.Exec(insertStmt, bank, from, to, value)
```

### 2. **Неправильные названия столбцов**
`from` и `to` - зарезервированные слова в SQL

**Исправление:**
```sql
-- Переименовать столбцы
from_currency VARCHAR
to_currency VARCHAR
```

## Серьезные проблемы

### 3. **Отсутствие обработки ошибок**
```go
req, _ := http.NewRequest(http.MethodGet, url.url, nil) // Игнорируется ошибка
body, _ := io.ReadAll(resp.Body) // Игнорируется ошибка
```

**Исправление:**
```go
req, err := http.NewRequest(http.MethodGet, url.url, nil)
if err != nil {
    return fmt.Errorf("creating request: %w", err)
}

body, err := io.ReadAll(resp.Body)
if err != nil {
    return fmt.Errorf("reading response: %w", err)
}
```

### 4. **Паника вместо обработки ошибок**
```go
if err != nil {
    panic(err) // Не используйте panic в production коде
}
```

**Исправление:**
```go
if err != nil {
    log.Printf("Error: %v", err)
    return err
}
```

## Улучшения кода

### 5. **Повторное подключение к БД**
```go
// Подключение к БД происходит для каждого запроса
func updateCurrency(...) error {
    db, err := sql.Open(...) // Открываем соединение
    defer db.Close()         // Закрываем соединение
}
```

**Исправление:**
```go
// Создать глобальное подключение или передавать его как параметр
var db *sql.DB

func initDB() error {
    psqlconn := fmt.Sprintf(...)
    var err error
    db, err = sql.Open("postgres", psqlconn)
    return err
}
```

### 6. **Магические числа и строки**
```go
if url.bankName == "Bank_2" { // Хардкод
```

**Исправление:**
```go
const (
    Bank1 = "Bank_1"
    Bank2 = "Bank_2"
)

if url.bankName == Bank2 {
```

### 7. **Улучшение структуры данных**
```go
urlsBank := []struct {
    bankName string
    curFrom  string
    curTo    string
    url      string
}{
```

**Исправление:**
```go
type BankConfig struct {
    Name    string
    From    string
    To      string
    URL     string
    Headers map[string]string
}

banks := []BankConfig{
    {
        Name: "Bank_1",
        From: "RUB",
        To:   "USD",
        URL:  "https://bank.example.com/rates/rub-usd",
    },
}
```

## Второстепенные улучшения

### 8. **Контекст и таймауты**
```go
clientBank := &http.Client{} // Без таймаутов
```

**Исправление:**
```go
clientBank := &http.Client{
    Timeout: 30 * time.Second,
}
```

### 9. **Логирование**
```go
fmt.Println("Connected!") // Использовать стандартный логгер
```

**Исправление:**
```go
log.Println("Connected to database")
```

### 10. **Валидация аргументов**
```go
if len(os.Args) == 2 {
    cmdName := os.Args[1]
    // ...
}
```

**Исправление:**
```go
if len(os.Args) != 2 {
    fmt.Println("Usage: ./currency <command>")
    os.Exit(1)
}
```

## Итоговый рекомендации

1. **Срочно исправить SQL injection**
2. **Добавить правильную обработку ошибок**
3. **Использовать подготовленные SQL выражения**
4. **Вынести подключение к БД из цикла**
5. **Добавить контекст и таймауты**
6. **Использовать структурированные конфиги для банков**

```go

package main

import (
	"context"
	"database/sql"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
	"time"

	_ "github.com/lib/pq"
)

// CurrencyRate представляет курс валют
type CurrencyRate struct {
	Bank      string
	From      string
	To        string
	Value     float64
	Timestamp time.Time
}

// BankConfig конфигурация банка
type BankConfig struct {
	Name     string
	Protocol string
	From     string
	To       string
	URL      string
	Headers  map[string]string
}

// RateStrategy интерфейс стратегии для получения курсов
type RateStrategy interface {
	GetRate(ctx context.Context, config BankConfig) (float64, error)
}

// StorageStrategy интерфейс стратегии для сохранения данных
type StorageStrategy interface {
	SaveRate(ctx context.Context, rate CurrencyRate) error
	Close() error
}

// HTTPRateStrategy стратегия для HTTP API
type HTTPRateStrategy struct {
	client *http.Client
}

func NewHTTPRateStrategy() *HTTPRateStrategy {
	return &HTTPRateStrategy{
		client: &http.Client{
			Timeout: 30 * time.Second,
		},
	}
}

func (s *HTTPRateStrategy) GetRate(ctx context.Context, config BankConfig) (float64, error) {
	req, err := http.NewRequestWithContext(ctx, "GET", config.URL, nil)
	if err != nil {
		return 0, fmt.Errorf("creating request: %w", err)
	}

	for key, value := range config.Headers {
		req.Header.Add(key, value)
	}

	resp, err := s.client.Do(req)
	if err != nil {
		return 0, fmt.Errorf("making request: %w", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return 0, fmt.Errorf("HTTP status %d", resp.StatusCode)
	}

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		return 0, fmt.Errorf("reading response: %w", err)
	}

	return s.parseResponse(body, config.Name)
}

func (s *HTTPRateStrategy) parseResponse(body []byte, bankName string) (float64, error) {
	// Простой парсинг - можно расширить для разных форматов
	strBody := string(body)
	
	if bankName == "Bank_1" {
		strBody = strings.ReplaceAll(strBody, ",", ".")
	}

	value, err := strconv.ParseFloat(strings.TrimSpace(strBody), 64)
	if err != nil {
		return 0, fmt.Errorf("parsing response: %w", err)
	}

	return value, nil
}

// JSONHTTPRateStrategy стратегия для HTTP API с JSON ответом
type JSONHTTPRateStrategy struct {
	client *http.Client
}

func NewJSONHTTPRateStrategy() *JSONHTTPRateStrategy {
	return &JSONHTTPRateStrategy{
		client: &http.Client{
			Timeout: 30 * time.Second,
		},
	}
}

func (s *JSONHTTPRateStrategy) GetRate(ctx context.Context, config BankConfig) (float64, error) {
	req, err := http.NewRequestWithContext(ctx, "GET", config.URL, nil)
	if err != nil {
		return 0, fmt.Errorf("creating request: %w", err)
	}

	for key, value := range config.Headers {
		req.Header.Add(key, value)
	}

	resp, err := s.client.Do(req)
	if err != nil {
		return 0, fmt.Errorf("making request: %w", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return 0, fmt.Errorf("HTTP status %d", resp.StatusCode)
	}

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		return 0, fmt.Errorf("reading response: %w", err)
	}

	var result struct {
		Rate float64 `json:"rate"`
	}
	if err := json.Unmarshal(body, &result); err != nil {
		return 0, fmt.Errorf("parsing JSON: %w", err)
	}

	return result.Rate, nil
}

// PostgresStorageStrategy стратегия для PostgreSQL
type PostgresStorageStrategy struct {
	db *sql.DB
}

func NewPostgresStorageStrategy(connStr string) (*PostgresStorageStrategy, error) {
	db, err := sql.Open("postgres", connStr)
	if err != nil {
		return nil, fmt.Errorf("opening database: %w", err)
	}

	if err := db.Ping(); err != nil {
		return nil, fmt.Errorf("connecting to database: %w", err)
	}

	return &PostgresStorageStrategy{db: db}, nil
}

func (s *PostgresStorageStrategy) SaveRate(ctx context.Context, rate CurrencyRate) error {
	query := `INSERT INTO currency_rates (bank, from_currency, to_currency, value, timestamp) 
	          VALUES ($1, $2, $3, $4, $5)`
	_, err := s.db.ExecContext(ctx, query, 
		rate.Bank, rate.From, rate.To, rate.Value, rate.Timestamp)
	return err
}

func (s *PostgresStorageStrategy) Close() error {
	return s.db.Close()
}

// FileStorageStrategy стратегия для сохранения в файл
type FileStorageStrategy struct {
	filePath string
}

func NewFileStorageStrategy(filePath string) *FileStorageStrategy {
	return &FileStorageStrategy{filePath: filePath}
}

func (s *FileStorageStrategy) SaveRate(ctx context.Context, rate CurrencyRate) error {
	file, err := os.OpenFile(s.filePath, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		return fmt.Errorf("opening file: %w", err)
	}
	defer file.Close()

	line := fmt.Sprintf("%s,%s,%s,%.4f,%s\n",
		rate.Bank, rate.From, rate.To, rate.Value, 
		rate.Timestamp.Format(time.RFC3339))
	
	if _, err := file.WriteString(line); err != nil {
		return fmt.Errorf("writing to file: %w", err)
	}

	return nil
}

func (s *FileStorageStrategy) Close() error { return nil }

// RateStrategyFactory фабрика для создания стратегий получения курсов
type RateStrategyFactory struct {
	strategies map[string]RateStrategy
}

func NewRateStrategyFactory() *RateStrategyFactory {
	return &RateStrategyFactory{
		strategies: map[string]RateStrategy{
			"http":      NewHTTPRateStrategy(),
			"http-json": NewJSONHTTPRateStrategy(),
			// Можно добавить другие стратегии: grpc, graphql и т.д.
		},
	}
}

func (f *RateStrategyFactory) GetStrategy(protocol string) (RateStrategy, error) {
	strategy, exists := f.strategies[protocol]
	if !exists {
		return nil, fmt.Errorf("unsupported protocol: %s", protocol)
	}
	return strategy, nil
}

// StorageStrategyFactory фабрика для создания стратегий хранения
type StorageStrategyFactory struct{}

func (f *StorageStrategyFactory) GetStrategy(storageType, connStr string) (StorageStrategy, error) {
	switch storageType {
	case "postgres":
		return NewPostgresStorageStrategy(connStr)
	case "file":
		return NewFileStorageStrategy(connStr), nil
	default:
		return nil, fmt.Errorf("unsupported storage type: %s", storageType)
	}
}

// App основное приложение
type App struct {
	rateStrategy    RateStrategy
	storageStrategy StorageStrategy
}

func NewApp(rateStrategy RateStrategy, storageStrategy StorageStrategy) *App {
	return &App{
		rateStrategy:    rateStrategy,
		storageStrategy: storageStrategy,
	}
}

func (a *App) UpdateRates(ctx context.Context, banks []BankConfig) error {
	for _, bank := range banks {
		rate, err := a.rateStrategy.GetRate(ctx, bank)
		if err != nil {
			log.Printf("Failed to get rate from %s: %v", bank.Name, err)
			continue
		}

		currencyRate := CurrencyRate{
			Bank:      bank.Name,
			From:      bank.From,
			To:        bank.To,
			Value:     rate,
			Timestamp: time.Now(),
		}

		if err := a.storageStrategy.SaveRate(ctx, currencyRate); err != nil {
			log.Printf("Failed to save rate from %s: %v", bank.Name, err)
			continue
		}

		log.Printf("Successfully updated rate from %s: %.4f %s/%s", 
			bank.Name, rate, bank.From, bank.To)
	}
	return nil
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ./currency update")
		os.Exit(1)
	}

	cmdName := os.Args[1]
	if cmdName != "update" {
		fmt.Println("Usage: ./currency update")
		os.Exit(1)
	}

	// Конфигурация банков
	banks := []BankConfig{
		{
			Name:     "Bank_1",
			Protocol: "http",
			From:     "RUB",
			To:       "USD",
			URL:      "https://bank.example.com/rates/rub-usd",
		},
		{
			Name:     "Bank_2",
			Protocol: "http-json",
			From:     "RUB",
			To:       "USD",
			URL:      "https://bank2.example.com/rates?from=RUB&to=USD",
			Headers:  map[string]string{"Authorization": "Bearer token123"},
		},
	}

	// Инициализация фабрик
	rateFactory := NewRateStrategyFactory()
	storageFactory := &StorageStrategyFactory{}

	// Для простоты используем первую стратегию из банков
	rateStrategy, err := rateFactory.GetStrategy(banks[0].Protocol)
	if err != nil {
		log.Fatalf("Rate strategy error: %v", err)
	}

	// Конфигурация хранилища (можно вынести в конфиг файл)
	storageStrategy, err := storageFactory.GetStrategy("postgres", 
		"host=localhost port=5432 user=postgres password=postgres dbname=currency sslmode=disable")
	if err != nil {
		log.Fatalf("Storage strategy error: %v", err)
	}
	defer storageStrategy.Close()

	app := NewApp(rateStrategy, storageStrategy)

	ctx, cancel := context.WithTimeout(context.Background(), 60*time.Second)
	defer cancel()

	if err := app.UpdateRates(ctx, banks); err != nil {
		log.Fatalf("Update rates failed: %v", err)
	}

	log.Println("Currency rates updated successfully")
}

```

## 3

```go
# Задача 3

| User           | OrderService    | AnalyticsService |
|----------------|-----------------|------------------|
| CreateOrder    |                 |                  |
|                | TrackOrder      |                  |
|                |                 |                  |
|                |                 | Too long action  |
|                |                 |                  |
|                |                 |                  |
```

**Решение проблемы потери заказов из-за медленного AnalyticsService**

# Вар 1
## Проблема
Синхронный вызов AnalyticsService блокирует основной поток создания заказа. Если аналитика работает медленно или недоступна, пользователь не получает ответ вовремя, что приводит к потере заказов и денег.

## Стратегия решения
**Разделение ответственности:** отделить критически важный процесс создания заказа от не-critical аналитики.

## Конкретные решения

### 1. **Асинхронная обработка аналитики**
- **Как:** После успешного создания заказа сразу отвечаем пользователю, а аналитику отправляем в фоновом режиме
- **Преимущество:** Пользователь не ждет завершения аналитики
- **Риск:** Потеря данных аналитики при падении сервиса

### 2. **Внедрение очереди сообщений**
- **Как:** OrderService публикует событие "заказ создан" в очередь (Kafka/RabbitMQ)
- **AnalyticsService** асинхронно consumes события из очереди
- **Преимущество:** Полная decoupling сервисов, буферизация нагрузки

### 3. **Circuit Breaker паттерн**
- **Как:** Автоматическое отключение вызовов к AnalyticsService при частых ошибках
- **Преимущество:** Защита OrderService от cascading failures
- **Как работает:** После N ошибок временно прекращаем вызовы, периодически проверяем восстановление

### 4. **Локальное хранение событий**
- **Как:** При недоступности AnalyticsService сохраняем события локально (БД, файл)
- **Преимущество:** Гарантированное сохранение данных даже при длительной недоступности аналитики
- **Восстановление:** Фоновая job периодически пытается отправить накопленные события

### 5. **Batch processing**
- **Как:** Накопление событий и отправка пачками раз в N минут
- **Преимущество:** Уменьшение нагрузки на AnalyticsService, более эффективное использование ресурсов

### 6. **Exponential backoff retry**
- **Как:** При ошибках увеличиваем интервалы между повторными попытками (100ms → 1s → 5s)
- **Преимущество:** Не заваливаем AnalyticsService запросами при восстановлении

## Рекомендуемая архитектура

1. **Синхронно:** Создание заказа → сохранение в основную БД → мгновенный ответ пользователю
2. **Асинхронно:** Публикация события в очередь → фоновая обработка AnalyticsService
3. **Защита:** Circuit breaker + retry mechanism + локальное fallback хранилище
4. **Мониторинг:** Alerting при длительной недоступности аналитики

## Результат
- ✅ Пользователи получают мгновенный ответ
- ✅ Заказы не теряются из-за проблем с аналитикой  
- ✅ AnalyticsService защищен от перегрузок
- ✅ Данные аналитики eventually consistent
- ✅ Система отказоустойчива к временной недоступности аналитики

# Вар 2

```go
# Задача 3

| User           | OrderService    | AnalyticsService |
|----------------|-----------------|------------------|
| CreateOrder    |                 |                  |
|                |                 |                  |
|                | TrackOrder      |                  |
|                |                 |                  |
| action         |                 | Too long         |

На примере создания заказа. Есть запрос на сервис и есть ответ, между этими двумя действиями мы отправляем данные в аналитику товара, которые заказали (например для подсчета популярности товаров). Сервис аналитики периодически работает медленно или вовсе таймаутит, и мы не успеваем ответить, теряем заказы.

Что делать, чтобы перестать терять заказы, и деньги соответственно?
```

## Решение проблемы

### 1. **Асинхронная обработка аналитики**
```go
// Вместо синхронного вызова AnalyticsService
func CreateOrder(order Order) error {
    // Создание заказа в основной БД
    if err := orderService.Save(order); err != nil {
        return err
    }
    
    // Асинхронная отправка в аналитику
    go func() {
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        
        analyticsService.TrackOrder(ctx, order)
    }()
    
    return nil
}
```

### 2. **Использование очереди сообщений**
```go
// OrderService публикует событие в очередь
func CreateOrder(order Order) error {
    if err := orderService.Save(order); err != nil {
        return err
    }
    
    // Публикация события в RabbitMQ/Kafka
    message := OrderCreatedEvent{
        OrderID: order.ID,
        UserID:  order.UserID,
        Items:   order.Items,
    }
    
    messageQueue.Publish("orders.created", message)
    return nil
}

// Отдельный consumer для AnalyticsService
func AnalyticsConsumer() {
    for message := range messageQueue.Consume("orders.created") {
        var event OrderCreatedEvent
        json.Unmarshal(message.Body, &event)
        
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        analyticsService.TrackOrder(ctx, event)
        cancel()
    }
}
```

### 3. **Паттерн Circuit Breaker**
```go
// Защита AnalyticsService от перегрузки
var analyticsCircuit = gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name:        "AnalyticsService",
    Timeout:     10 * time.Second,
    MaxRequests: 5,
    Interval:    1 * time.Minute,
    ReadyToTrip: func(counts gobreaker.Counts) bool {
        return counts.ConsecutiveFailures > 5
    },
})

func TrackOrderWithCircuitBreaker(order Order) error {
    _, err := analyticsCircuit.Execute(func() (interface{}, error) {
        return nil, analyticsService.TrackOrder(order)
    })
    return err
}
```

### 4. **Кэширование и batch processing**
```go
// Накопление событий и пакетная обработка
type AnalyticsBuffer struct {
    events []OrderEvent
    mu     sync.Mutex
    ticker *time.Ticker
}

func (b *AnalyticsBuffer) Add(event OrderEvent) {
    b.mu.Lock()
    defer b.mu.Unlock()
    b.events = append(b.events, event)
}

func (b *AnalyticsBuffer) StartBatchProcessor() {
    for range b.ticker.C {
        b.mu.Lock()
        if len(b.events) > 0 {
            go b.processBatch(b.events)
            b.events = nil
        }
        b.mu.Unlock()
    }
}

func (b *AnalyticsBuffer) processBatch(events []OrderEvent) {
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    analyticsService.TrackOrdersBatch(ctx, events)
}
```

### 5. **Ретри механизм с экспоненциальной отсрочкой**
```go
func TrackOrderWithRetry(order Order) error {
    retryPolicy := retry.NewExponentialBackoff(
        retry.InitialInterval(100*time.Millisecond),
        retry.MaxInterval(5*time.Second),
        retry.MaxAttempts(3),
    )
    
    return retry.Do(func() error {
        return analyticsService.TrackOrder(order)
    }, retryPolicy)
}
```

### 6. **Локальное хранение при недоступности аналитики**
```go
func CreateOrder(order Order) error {
    if err := orderService.Save(order); err != nil {
        return err
    }
    
    // Сохранение события в локальную очередь для последующей отправки
    event := OrderEvent{
        OrderID: order.ID,
        Timestamp: time.Now(),
        Data: order,
    }
    
    if err := localQueue.Store(event); err != nil {
        log.Printf("Failed to store analytics event: %v", err)
        // Не прерываем основной flow!
    }
    
    return nil
}

// Фоновая горутина для обработки отложенных событий
func ProcessFailedEvents() {
    for {
        events := localQueue.GetFailedEvents(100) // batch по 100 событий
        if len(events) > 0 {
            if err := analyticsService.TrackOrdersBatch(events); err == nil {
                localQueue.RemoveEvents(events)
            }
        }
        time.Sleep(1 * time.Minute)
    }
}
```

## Рекомендуемая архитектура:

1. **Основной flow**: Синхронное создание заказа → мгновенный ответ пользователю
2. **Аналитика**: Асинхронная обработка через очередь сообщений
3. **Защита**: Circuit breaker + retry mechanism
4. **Надежность**: Локальное хранение неудачных событий
5. **Производительность**: Batch processing для аналитики

**Результат:** Заказы не теряются даже при полной недоступности AnalyticsService, пользователь получает мгновенный ответ, а аналитика обрабатывается когда это становится возможным.

# Вар 3

**Короткий флоу без изменений в AnalyticsService:**

1. **Получение запроса** → OrderService создает заказ в основной БД ✅

2. **Мгновенный ответ** → Пользователь получает подтверждение сразу ✅

3. **Асинхронная аналитика** → В фоне:
   - Сохраняем событие в локальную очередь (БД/Redis)
   - Отправляем в AnalyticsService без ожидания ответа
   - При ошибке - оставляем в очереди для повторных попыток

4. **Фоновая обработка** → Отдельный воркер:
   - Раз в минуту забирает события из очереди
   - Пытается отправить в AnalyticsService с retry
   - При успехе - удаляет из очереди

**Что это дает:**
- 🚀 Пользователь не ждет аналитику
- 💰 Не теряем заказы из-за медленной аналитики  
- 🛡️ OrderService защищен от таймаутов Analytics
- 📊 Данные аналитики все равно попадут в систему (eventually)
- 🔧 Не требуются изменения в AnalyticsService

# 4
**Условие задачи:**

Есть три сущности: пользователь, чат и сообщение.

**Требования к данным:**
- У пользователя есть имя и дата регистрации
- У чата есть название и дата создания  
- У сообщения есть текст, автор и дата создания
- Пользователь может состоять в нескольких чатах одновременно
- Сообщение обязательно принадлежит ровно одному чату
- Сообщение не может принадлежать более чем одному чату одновременно

**Задача:**
Описать предметную область в виде таблиц базы данных, обеспечив все указанные требования и связи между сущностями.

---

**SQL реализация:**

```sql
-- Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    registered_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица чатов
CREATE TABLE chats (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица сообщений
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    chat_id INTEGER NOT NULL REFERENCES chats(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    text TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица связи пользователей и чатов (многие-ко-многим)
CREATE TABLE user_chats (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    chat_id INTEGER NOT NULL REFERENCES chats(id) ON DELETE CASCADE,
    joined_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, chat_id)
);
```

**Проверка выполнения требований:**
1. ✅ Пользователь имеет имя и дату регистрации
2. ✅ Чат имеет название и дату создания
3. ✅ Сообщение имеет текст, автора и дату создания
4. ✅ Пользователь в нескольких чатах (через user_chats)
5. ✅ Сообщение принадлежит ровно одному чату (chat_id NOT NULL)
6. ✅ Сообщение не может быть в нескольких чатах (один chat_id на сообщение)


# 5
**Условие задачи:**

Есть три сущности: пользователь, чат и сообщение.

**Требования к данным:**
- У пользователя есть имя и дата регистрации
- У чата есть название и дата создания  
- У сообщения есть текст, автор и дата создания
- Пользователь может состоять в нескольких чатах одновременно
- Сообщение обязательно принадлежит ровно одному чату
- Сообщение не может принадлежать более чем одному чату одновременно

**Задача:**
Описать предметную область в виде таблиц базы данных, обеспечив все указанные требования и связи между сущностями.

---

**SQL реализация:**

```sql
-- Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    registered_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица чатов
CREATE TABLE chats (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица сообщений
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    chat_id INTEGER NOT NULL REFERENCES chats(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    text TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица связи пользователей и чатов (многие-ко-многим)
CREATE TABLE user_chats (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    chat_id INTEGER NOT NULL REFERENCES chats(id) ON DELETE CASCADE,
    joined_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, chat_id)
);
```

**Проверка выполнения требований:**
1. ✅ Пользователь имеет имя и дату регистрации
2. ✅ Чат имеет название и дату создания
3. ✅ Сообщение имеет текст, автора и дату создания
4. ✅ Пользователь в нескольких чатах (через user_chats)
5. ✅ Сообщение принадлежит ровно одному чату (chat_id NOT NULL)
6. ✅ Сообщение не может быть в нескольких чатах (один chat_id на сообщение)