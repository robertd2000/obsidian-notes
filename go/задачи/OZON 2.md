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

