```go

package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	semaphore := make(chan struct{}, 3) // семафор на 3 одновременных задачи

	for i := 1; i <= 10; i++ {
		wg.Add(1) // Добавляем задачу ДО запуска горутины
		go printNumber(i, &wg, semaphore)
	}

	wg.Wait()
}

func printNumber(n int, wg *sync.WaitGroup, semaphore chan struct{}) {
	// Захватываем семафор
	semaphore <- struct{}{}
	defer func() {
		<-semaphore // Освобождаем семафор при выходе
	}()

	time.Sleep(time.Second)
	fmt.Println(n)

	// Уменьшаем счётчик WaitGroup
	wg.Done()
}

```