// Есть слайс целых чисел, нужно написать функцию которая вернет слайс такого же размера,
// где на каждой i-ой позиции будет произведение всех элементов кроме i-го
// [1, 2, 3] => [2*3, 1*3, 1*2] => [6, 3, 2]

package main

import (
    "fmt"
)

func MultOther(in []int) []int {
    n := len(in)
    out := make([]int, n)
    
    pre := 1
    
    for i := 0; i < n; i++ {
        out[i] = pre
        pre *= in[i]
    }
    
    post := 1
    
    for i := n - 1; i >= 0; i-- {
        out[i] *= post
        post *= in[i]
    }
    
    return out
}

//----------------------------------------------------------------------

// Требуется обойти все url и получить слайс с кодами ответов в том же порядке
// Хотим делать параллельно но не больше k запросов одновременно

package main

import (
    "fmt"
    "math/rand"
    "net/http"
    "time"
)

var urls = []string{
    "https://www.lamoda.ru/p/mp002xw0lvkd/clothes-tomollyfromjames-plate/",
    "https://www.lamoda.ru/p/mp002xw14uf2/clothes-tomollyfromjames-plate/",
    "https://www.lamoda.ru/p/rtladr746901/clothes-iceberg-plate/",
    "https://www.lamoda.ru/p/mp002xw18h9d/clothes-victoriaveisbrut-plate/",
    "https://www.lamoda.ru/p/mp002xw004x4/clothes-clanvi-plate/",
    "https://www.lamoda.ru/p/mp002xw0zfxy/clothes-glvr-plate/",
    "https://www.lamoda.ru/p/mp002xw0slmg/clothes-snezhnayakoroleva-plate-kozhanoe/",
    "https://www.lamoda.ru/p/mp002xw132c3/clothes-auranna-plate/",
    ......
}

type urlItem struct {
    url string
    idx int
}

func crawl(urls []string, k int) []int {
    res := make([]int, len(urls))
    in := make(chan urlItem)
    cash := make(map[string]int)
    
    mx := sync.Mutex{}
    
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    go func() {
        defer close(in)
        for idx, url := range urls {
            in <- urlItem{
                url,
                idx
            }
            
        }
    }()
    
    wg := &sync.WaitGroup{}
    wg.Add(k)
    
    for range k {
        go func() {
            defer wg.Done()
            
            for {
                select {
                    case v, ok := <- in:
                        if !ok {
                            return
                        }
                        mx.Lock()
                        status, ok := cash[v.url]
                        mx.Unlock()
                        
                        if !ok {
                            status, _ = fetchUrl(ctx, v.url)
                            mx.Lock()
                            cash[v.url] = status
                            mx.Unlock()
                        }
                        
                        res[v.idx] = status
                }
            }
        }()
    }
    
    wg.Wait()
    
    return res
}

func fetchUrl(ctx context.Context, url string) (int, error) {
    req, err := http.NewRequestWithContext(ctx, url)
    if err != nil {
        return 0, err
    }
    
    resp, err = http.Do(req)
    status := resp.Body.StatusCode
    resp.Body.Close()
    return status, err
}

func main() {    
    result := crawl(urls, 5)    
    fmt.Println("All done")
} 

//--------------------------------------------------------------------

Структура:

CREATE TABLE customer
(
    id      INTEGER PRIMARY KEY,
    email   VARCHAR(100)    NOT NULL,
    country CHAR(2)         NOT NULL
);

CREATE TABLE cart_item
(
    id          INTEGER PRIMARY KEY,
    customer_id INTEGER         NOT NULL,
    title       VARCHAR(20)     NOT NULL,
    amount      INTEGER         NOT NULL,
    price       INTEGER         NOT NULL    
);

Пример данных:

customer
--------
1, c1@example.com, ru
2, c2@example.com, ru
3, c3@example.com, ru
4, c4@example.com, by

cart_item
---------
1,  1, пиво,    6, 100
2,  1, вода,    2,  50
3,  2, печенье, 1,  75
4,  2, сок,     2,  60

---Вывести построчно всех кастомеров (id, email) и все элементы корзины пользователя (title, amount)
select id, email, title, amount from customer
join cart_item on customer.id = cart_item.customer_id

---Вывести топ-10 клиентов (id, email) по общей стоимости товаров в корзине

select id, email, sum(price * amount) as total_price from customer
join cart_item on customer.id = cart_item.customer_id
group by customer.id, customer.email
having total_price > 1000
where country = 'ru'
order by total_price desc
limit 10;

------------------------------------------------------------

CREATE TABLE users(username text, password text, age integer);
CREATE INDEX h_idx ON users USING hash(age); 
SELECT * FROM users WHERE age > 18;

CREATE INDEX myIdx ON carts(sku, country, customer_id);
SELECT * FROM carts WHERE sku = 192 AND country = 'ru';
SELECT * FROM carts WHERE country = 'ru' AND customer_id = 10

-------------------------------------------------------------

-- Юзеры и их балансы
CREATE TABLE users
(
    user_id INTEGER PRIMARY KEY,
    balance REAL    NOT NULL
);

// Снятие денег с баланса пользователя
func withdrawBalance(userId int32, amount float32) float32 {
    tx := BeginTransaction("ISOLATION LEVEL READ COMMITTED")
        
    oldBalance := tx.exec("SELECT balance FROM users WHERE user_id = $1 for update", userId)
    if oldBalance >= amount  {
        tx.exec("UPDATE users SET balance = balance - $2 WHERE user_id = $1 and balance >= $2 returning balance", userId, amount)
    }
    newBalance := tx.exec("SELECT balance FROM users WHERE user_id = $1 ", userId)

    tx.Commit()
    
    return newBalance
}








