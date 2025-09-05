

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
)

// Необходимо поочередно выполнить HTTP-запросы по предложенному списку ссылок:

// - Если получен HTTP-код ответа "200 OK" - вывести "адрес url - ok"

// - Если получен любой другой HTTP-код ответа или произошла ошибка - вывести "адрес url - not ok"
// 2. Модифицируйте программу таким образом, чтобы использовать каналы для коммуникаций основного потока с горутинами. Пример:
// Запросы по списку выполняются в горутинах.
// Печать результатов на экран происходит в основном потоке
     //  3 Урлы приходят из внешнего источника, длина неизвестна
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
	urlsChan := make(chan string)
	parrallelCount := 5

	
	go func() {
		for _, url := range urls {
			urlsChan <- url
		}
		
		close(urlsChan)
	}()
	ch := make(chan string)
	wg := &sync.WaitGroup{}

	wg.Add(n)

	for i := 0; i < parrallelCount; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for _, url := range urlsChan {
				proccessUrl(url, ch)
			}
		}()
	}


	go func() {
		wg.Wait()
		close(ch)
	}()

	for v := range ch {
		fmt.Println(v)
	}
}

func proccessUrl(url string, ch chan <- string) {
	resp, err := http.Get(url)

	if err != nil {
		ch <- fmt.Sprintf("адрес %s - not ok", url )
		return
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		ch <- fmt.Sprintf("адрес %s - not ok", url)
	} else {
		ch <- fmt.Sprintf("адрес %s - ok", url)
	}
}

```