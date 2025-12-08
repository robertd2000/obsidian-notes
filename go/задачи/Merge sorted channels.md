```go
package main

import "fmt"

func gen(nums ...int) <-chan int {
	ch := make(chan int)
	go func() {
		defer close(ch)
		for _, n := range nums {
			ch <- n
		}
	}()
	return ch
}

func main() {
	ch1 := gen(1, 3, 5, 7)
	ch2 := gen(2, 4, 6, 8, 9, 10)

	merged := merge(ch1, ch2)

	for x := range merged {
		fmt.Print(x, " ")
	}
	// Вывод: 1 2 3 4 5 6 7 8 9 10
}

func merge(ch1, ch2 <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		// Предварительно читаем первые значения
		v1, ok1 := <-ch1
		v2, ok2 := <-ch2

		for ok1 && ok2 {
			if v1 <= v2 {
				out <- v1
				v1, ok1 = <-ch1
			} else {
				out <- v2
				v2, ok2 = <-ch2
			}
		}

		// Сливаем остаток из ch1
		for ok1 {
			out <- v1
			v1, ok1 = <-ch1
		}

		// Сливаем остаток из ch2
		for ok2 {
			out <- v2
			v2, ok2 = <-ch2
		}
	}()

	return out
}
```