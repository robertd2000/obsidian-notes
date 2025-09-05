

https://www.youtube.com/watch?v=-8JlOr3Z0eA

## 1

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

## 2

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

## 3

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

## 3

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