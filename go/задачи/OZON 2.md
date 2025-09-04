https://www.youtube.com/watch?v=f5lcPQNNidc
# 1

```go

package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Println(uniqRandn(10))
}

// Функция uniqRandn генерирует слайс длины n из уникальных случайных чисел
func uniqRandn(n int) []int {
	res := make([]int,0, n)
	set := make(map[int]bool)
	
	for len(res) < n {
		num := rand.Intn(n * 2)
		
		if !set[num] {
			res = append(num)
			set[num] = true
		}		
	}
	
	return res
}
```