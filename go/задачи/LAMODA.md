
# 1

```go

package main

import "fmt"

func foo(arr []int) {
    arr = append(arr, 9, 9)
}

func main() {
    src := []int{1, 2, 3, 4, 5}
    arr := src[:3] 

    foo(arr)

    fmt.Println(src) // 1, 2, 3, 9, 9
    fmt.Println(arr) // 1, 2, 3
}
```

1. **Исходный массив**: `src` имеет длину 5 и емкость 5
    
2. **Создание среза**: `arr := src[:3]` создает срез:
    
    - Данные: `[1, 2, 3]`
        
    - Длина: 3
        
    - **Емкость: 5** (потому что берется из исходного массива)
        
3. **В функции foo**:
    
    - `append(arr, 9, 9)` пытается добавить два элемента
        
    - Поскольку емкость (5) > текущей длины (3) + новые элементы (2),  
        **не происходит переаллокации памяти**
        
    - Новые элементы записываются в исходный массив `src` на позиции 4 и 5
        
4. **Но почему arr не изменился?**
    
    - В Go срезы передаются по значению (копируется заголовок среза)
        
    - Функция `foo` получает копию заголовка среза
        
    - `append` изменяет локальную копию, но не оригинальный срез `arr` в main
        
    - Однако данные записываются в общий базовый массив

## как не мутирровать изначальный слайс?

1.
```go

	arr := src[:3:3]

```

если сделать так, то у arr cap будет 3 по формуле 

```
For slice[i:j:k] or [2:3:4]
Length: j - i or 3 - 2 = 1
Capacity: k - i or 4 - 2 = 2
```

и значит при append т.к. capacity переполнится создастся новый массив

2.
можно изменить foo, чтобы возвращал слайс

3 можно копию сделать

# 2

```go

package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
)

func main() {
	urls := []string{
		"https://www.lamoda.ru",
		"https://www.yandex.ru",
		"https://www.mail.ru",
		"https://www.google.com",
	}
	
	for _, url := range urls {
		go func(url string) {
			fmt.Printf("Fetching %s...\n", url)
			err := fetchUrl(context.Background(), url)
			if err != nil {
				fmt.Printf("Error fetching %s: %v\n", url, err)
				return
			}
			fmt.Printf("Fetched %s\n", url)
		}(url)
	}

	fmt.Println("All requests launched!")
	time.Sleep(400 * time.Millisecond) // проблема - лучше WaitGroup{}
	fmt.Println("Program finished.")
}

func fetchUrl(ctx context.Context, url string) error {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return err
	}
	_, err = http.DefaultClient.Do(req)
	return err
}

```

```go

package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
	"sync"
	"context"
)

func main() {
	urls := []string{
		"https://www.lamoda.ru",
		"https://www.yandex.ru",
		"https://www.mail.ru",
		"https://www.google.com",
	}
	
	var wg &sync.WaitGroup{}
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	
	for _, url := range urls {
		wg.Add(1)
		go func(url string) {
			defer wg.Done()
			fmt.Printf("Fetching %s...\n", url)
			
			err := fetchUrl(ctx, url)
			if err != nil {
				cancel()
				fmt.Printf("Error fetching %s: %v\n", url, err)
				return
			}
			fmt.Printf("Fetched %s\n", url)
		}(url)
	}

	fmt.Println("All requests launched!")
	wg.Wait()
	fmt.Println("Program finished.")
}

func fetchUrl(ctx context.Context, url string) error {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return err
	}
	_, err = http.DefaultClient.Do(req)
	return err
}

```

```go

package main

import (
	"context"
	"fmt"
	"net/http"
	"sync"
	"time"
)

func main() {
	urls := []string{
		"https://www.lamoda.ru",
		"https://www.yandex.ru",
		"https://www.mail.ru",
		"https://www.google.com",
	}

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	var wg sync.WaitGroup
	results := make(chan string, len(urls))
	errCh := make(chan error, len(urls))

	for _, url := range urls {
		wg.Add(1)
		go func(url string) {
			defer wg.Done()
			
			fmt.Printf("Fetching %s...\n", url)
			err := fetchUrl(ctx, url)
			if err != nil {
				errCh <- fmt.Errorf("error fetching %s: %v", url, err)
				return
			}
			results <- fmt.Sprintf("Fetched %s", url)
		}(url)
	}

	go func() {
		wg.Wait()
		close(results)
		close(errCh)
	}()

	fmt.Println("All requests launched!")

	// Обработка результатов
	for result := range results {
		fmt.Println(result)
	}

	// Обработка ошибок
	for err := range errCh {
		fmt.Println(err)
	}

	fmt.Println("Program finished.")
}

func fetchUrl(ctx context.Context, url string) error {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return err
	}
	_, err = http.DefaultClient.Do(req)
	return err
}

```