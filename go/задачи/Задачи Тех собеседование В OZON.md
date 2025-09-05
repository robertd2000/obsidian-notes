

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