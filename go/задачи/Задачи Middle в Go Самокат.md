
## Merge channels

```go

// нужно слить n каналов в один - смержить все данные в один и вернет их
package main

import (
	"fmt"
	"sync"
)

func merge(chans ...chan int) <- chan int {
	out := make(chan int)
	
	wg := &sync.WaitGroup{}
	
	wg.Add(len(chans))
	
	for _, ch := range chans {
		go func(ch <-chan int, wg *sync.WaitGrou) {
			defer wg.Done()
			for c := range ch {
				out <- c
			}
		}(ch, wg)
	}
	
	go func() {
		wg.Wait()
		close(out)
	}()
	
	return out
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	ch3 := make(chan int)
	
	go func() {
		for i := 0; i < 10; i++ {
			ch1 <- i
		}
		close(ch1)
	}() 
		
	go func() {
		for i := 10; i < 20; i++ {
			ch2 <- i
		}
		close(ch2)
	}() 
		
	go func() {
		for i := 20; i < 30; i++ {
			ch3 <- i
		}
		close(ch3)
	}() 
	
	m := merge(ch1, ch2, ch3)
	
	for c := range m {
		fmt.Println(c)
	}
}

```

Написать воркер пулл с заданной функцией

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, f func(int) int, jobs <-chan int, res chan<- int) {
	for job := range jobs {
		res <- f(job)
	}
}

func square(n int) int {
	return n * n
}

func main() {
	workersNums := 4
	
	wg := sync.WaitGroup{}
	wg.Add(workersNums)
	
	jobs := make(chan int, 10)
	res := make(chan int, 10)
	
	for i := 0; i < 10; i++ {
		jobs <- i
	}
	close(jobs)
	
	for i := range workersNums {
		go func() {
			defer wg.Done()
			worker(i, square, jobs, res)
		}() 
	}
	
	go func() {
		wg.Wait()
		close(res)
	}()
	
	for v := range res {
		fmt.Println(v)
	}
}

```

