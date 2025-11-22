```go

package main

import (
    "fmt"
    "sync"
)

func main() {
    fmt.Println(GetOrCreate("hello", "world"))
    fmt.Println(Get("hello"))
}


var cache = make(map[string]string)
var m sync.RWMutex
// GetOrCreate проверяет существование ключа key
// Если такого нет, то создает новое значение
func GetOrCreate(key, value string) string {
    m.RLock()
    cashValue, ok = cache[key]
    m.RUnlock() 

    if ok {
        return cashValue
    }

    m.Lock()
    cache[key] = value
    m.Unlock() 
    
    return value
}

func Get(key string) string {
    m.RLock()
    v := cache[key]
    m.RUnlock()
    return v
}

```