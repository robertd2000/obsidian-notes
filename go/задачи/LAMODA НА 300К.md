```go

func foo(src []int) {
	src = append(src, 5)
}

func main() {
	arr := []int{1, 2, 3}
	src := arr[:1]
	
	foo(src)
	fmt.Println(arr) // 1 5 3
	fmt.Println(src) // 1
}
```

```go
package main

import (
	"fmt",
	"time"
)
var m = map[string]int{"a": 1}

func main() {
	go read()
	time.Sleep(1 * time.Second)
	go write()
	time.Sleep(1 * time.Second)
}

func read() {
	for {
		fmt.Println(m["a"])	
	}
}

func write() {
	for {
		m["a"] = 2	
	}
}
// после 1 сек паника после записи
// бесконечный цикл
```

```go
package main

import (
	"fmt",
	"time"
)

func main() {
	ch := make(chan int)
	
	go func() {
		ch <- 1
		close(ch)
	}()
	
	time.Sleep(500*time.Millisecond)
	
	
	
	for i := range ch {
		fmt.Println(i)
	}
	
	time.Sleep(100*time.Millisecond)
}

```

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
    "sync"
)

func merge(ch ...<-chan int) <-chan int {
    out := make(chan int)

    // Имеется 2 входных канала in1 и in2 и один выходной out.
    // Требуется реализовать функцию merge, которая будет сливать данные из входных каналов в один выходной.
    wg := &sync.WaitGroup{}
    for _, c := range ch {
	    wg.Add(1)
	    go func(c <-chan int) {
		    defer wg.Done()
		    for i := range c {
			    out <- i
		    }
	    }(c) 
    }
    
    go func() {
	    wg.Wait()
	    close(out)
    }()

    return out
}

func source(sourceFunc func(int) int) <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)
        for i := 0; i < 10; i++ {
            ch <- sourceFunc(i)
            time.Sleep(time.Duration(rand.Intn(3)) * time.Second)
        }
    }()

    return ch
}

func main() {
    rand.Seed(time.Now().UnixNano())
    in1 := source(func(i int) int {
        return rand.Int()
    })
    in2 := source(func(i int) int {
        return i
    })
    out := merge(in1, in2)

    for val := range out {
        fmt.Println("Value: ", val)
    }
}
```

```go
package main

import "fmt"

type MyError struct {
    data string
}

func (e MyError) Error() string {
    return e.data
}

func main() {
    err := foo(4)
    if err != nil {
        fmt.Println("oops")
    } else {
        fmt.Println("ok")
    }
}

func foo(i int) error {
    var err *MyError
    if i > 5 {
        err = &MyError{data: "i > 5"}
    }

    return err
} 
//  oops
```

![[Pasted image 20251110183356.png]]

![[Pasted image 20251110183405.png]]

-- Выбираем построчно всех покупателей (id, email) и все элементы корзины пользователя (sku, amount)
```sql
SELECT 
    customer.id,
    customer.email,
    carts.sku,
    carts.amount
FROM public.customer
JOIN public.carts ON public.customer.id = public.carts.customer_id;
```

-- Выбираем топ-10 покупателей (id, email) по количеству элементов в корзине (с учётом amount)
```sql
SELECT 
    public.customer.id,
    public.customer.email,
    SUM(public.carts.amount) as total_items
FROM public.customer
JOIN public.carts ON public.customer.id = public.carts.customer_id
GROUP BY public.customer.id, public.customer.email
ORDER BY total_items DESC
LIMIT 10;
```

![[Pasted image 20251110212156.png]]


