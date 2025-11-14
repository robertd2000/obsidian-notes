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
	set := make(map[int]struct{})
	
	for len(res) < n {
        num := rand.Intn(n * 2)
        
        if _, exists := set[num]; !exists {
            res = append(res, num) // исправлено здесь
            set[num] = struct{}{}
        }        
    }
	
	return res
}

```

```go

package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	fmt.Println(uniqRandn(3))
}

// Оптимизированная функция uniqRandn с минимальными аллокациями памяти
func uniqRandn(n int) []int {
	if n <= 0 {
		return nil
	}

	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	
	// Единственная аллокация - сразу создаем срез нужного размера
	result := make([]int, n)
	
	// Используем map с предварительным выделением емкости для уменьшения реаллокаций
	unique := make(map[int]struct{}, n)
	
	i := 0
	for i < n {
		num := r.Int()
		
		// Проверяем наличие через пустую структуру (0 байт)
		if _, exists := unique[num]; !exists {
			unique[num] = struct{}{} // struct{}{} занимает 0 байт
			result[i] = num
			i++
		}
	}
	
	return result
}

```

# 2 

```sql

Таблица "orders_1"

| id | user_id | created_at          | price_total |
|----|---------|---------------------|-------------|
| 1  | 111     | 2024-01-01 10:00:08 | 2989        |
| 2  | 222     | 2024-01-02 10:00:08 | 180         |
| 3  | 111     | 2024-04-01 10:00:08 | 20980       |
| 4  | 222     | 2024-05-01 10:00:08 | 5980        |
| 5  | 333     | 2024-05-02 10:00:08 | 10080       |

Запрос, который вернет количество заказов по каждому пользователю с price_total больше или равным 1800, отсортированному по количеству заказов в обратном порядке:

select 
	user_id, 
	count(*) as order_count
from orders_1
where price_total >= 1800
group by user_id
order by order_count desc;

```

# 3

```go

n := map[string]int{"a": 1, "b": 2, "c": 3}

for a, b := range n {
	fmt.Println(a, b)
}

// a 1 b 2 c 3 в случайном порядке

fmt.Println(n) // в отсортированном по ключам
```

# 4

```go

func main() {
    nums := []int{1, 2, 3}
    addNum(nums[0:2])
    fmt.Println(nums) // ? [1 2 4] // так как cap не переполнился, значит перезапишется
    addNums(nums[0:2]) 
    fmt.Println(nums) // ? [1 2 4] - тут присвоится новый массив, поэтому nums не изменится, тк в функцию передается по значению
}

func addNum(nums []int) {
    nums = append(nums, 4)
}

func addNums(nums []int) {
    nums = append(nums, 5, 6)
}
 
```