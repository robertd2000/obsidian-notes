https://www.youtube.com/watch?v=mB13FSDzcGo

# sync/singleflight - https://habr.com/ru/articles/677444/
# 1
```go

package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"sync"
	"time"
)

type WeatherService struct {
	Temperature int
	mu           sync.RWMutex
}

func NewWeatherService(interval time.Duration) *WeatherService {
	ws := &WeatherService{}


	ticker := time.NewTicker(interval)

	go func() {
		defer ticker.Stop()

		for range ticker.C {
			ws.UpdateTemperature()
		}
	}()

	return ws
}

func (d *WeatherService) UpdateTemperature() {
	tmp := WeatherForecast()
	d.mu.Lock()
	defer d.mu.Unlock()
	d.Temperature = tmp
}

func (d *WeatherService) GetTemperature() int {
	d.mu.RLock()
	defer d.mu.RUnlock()

	return d.Temperature
}

func WeatherForecast(city string) int {
	time.Sleep(1 * time.Second)
	return rand.Intn(70) - 30
}

func main() {
initialCities := []string{"Moscow", "London", "New York", "Tokyo", "Berlin"}
	data := NewWeatherService(1 * time.Minute)
		fmt.Println(w, "{\"temperature\":%d}\n", data.GetTemperature())
	})

	if err := http.ListenAndServe(":3333", nil); err != nil {
		panic(err)
	}
}
```
# 2
```go

package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"sync"
	"time"
)

type WeatherService struct {
	Temperatures map[string]int
	mu           sync.RWMutex
}

func NewWeatherService(interval time.Duration) *WeatherService {
	ws := &WeatherService{
		Temperatures: make(map[string]int, len(cities)),
	}
	
	ticker := time.NewTicker(interval)

	go func() {
		defer ticker.Stop()

		for range ticker.C {
			ws.UpdateTemperature()
		}
	}()

	return ws
}

func (d *WeatherService) UpdateTemperature() {
	wg := &sync.WaitGroup{}
	for city := range d.Temperatures {
		wg.Add(1)
		go func(c string) {
			defer wg.Done()

			tmp := WeatherForecast(city)

			d.mu.Lock()
			d.Temperatures[city] = tmp
			d.mu.Unlock()
		}(city)
	}

	wg.Wait()
}

func (d *WeatherService) GetTemperature(city string) (int, error) {
	d.mu.RLock()
	defer d.mu.RUnlock()

	t, ok := d.Temperatures[city]
	if !ok {
		return 0, fmt.Errorf("city %s not found", city)
	}
	return t, nil
}

func WeatherForecast(city string) int {
	time.Sleep(1 * time.Second)
	return rand.Intn(70) - 30
}

func main() {
	data := NewWeatherService(initialCities, 1 * time.Minute)

	http.HandleFunc("/weather", func(w http.ResponseWriter, r *http.Request) {
		city := r.URL.Query().Get("city")
		if city == "" {
			http.Error(w, `{"error": "city parameter is required"}`, http.StatusBadRequest)
			return
		}
	
		temp, err := data.GetTemperature(city)
		if err != nil {
			http.Error(w, err.Error(), http.StatusNoContent)
		}

		fmt.Println(w, "{\"temperature\":%d}\n", temp)
	})

	if err := http.ListenAndServe(":3333", nil); err != nil {
		panic(err)
	}
}

```

# 3 

```go

package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"slices"
	"sync"
	"time"
)

type WeatherService struct {
	temperatures map[string]int
	mu           sync.RWMutex
	cities       []string
}

func NewWeatherService(cities []string, interval time.Duration) *WeatherService {
	ws := &WeatherService{
		temperatures: make(map[string]int, len(cities)),
		cities:       cities,
	}

	for _, city := range cities {
		ws.temperatures[city] = WeatherForecast(city)
	}

	ticker := time.NewTicker(interval)

	go func() {
		defer ticker.Stop()

		for range ticker.C {
			ws.UpdateTemperature()
		}
	}()

	return ws
}

func (ws *WeatherService) UpdateTemperature() {
	ws.mu.RLock()
	cities := make([]string, len(ws.cities))
	copy(cities, ws.cities)
	ws.mu.RUnlock()

	wg := &sync.WaitGroup{}
	for _, city := range cities {
		wg.Add(1)
		go func(c string) {
			defer wg.Done()

			tmp := WeatherForecast(c)

			ws.mu.Lock()
			ws.temperatures[city] = tmp
			ws.mu.Unlock()
		}(city)
	}

	wg.Wait()
}

func (ws *WeatherService) GetTemperature(city string) (int, bool) {
	ws.mu.RLock()
	defer ws.mu.RUnlock()

	t, ok := ws.temperatures[city]

	return t, ok
}

func (ws *WeatherService) updateTemperature(city string) {
	temp := WeatherForecast(city)

	ws.mu.Lock()
	ws.temperatures[city] = temp
	ws.mu.Unlock()
}

func (ws *WeatherService) AddCity(city string) {
	ws.mu.Lock()

	if slices.Contains(ws.cities, city) {
		defer ws.mu.Unlock()
		return
	}

	ws.cities = append(ws.cities, city)
	ws.temperatures[city] = 0
	defer ws.mu.Unlock()

	go ws.updateTemperature(city)
}

func WeatherForecast(city string) int {
	time.Sleep(1 * time.Second)
	return rand.Intn(70) - 30
}

func main() {
	initialCities := []string{"Moscow", "London", "New York", "Tokyo", "Berlin"}
	weatherService := NewWeatherService(initialCities, 1*time.Minute)

	http.HandleFunc("/weather", func(w http.ResponseWriter, r *http.Request) {
		city := r.URL.Query().Get("city")
		if city == "" {
			http.Error(w, `{"error": "city parameter is required"}`, http.StatusBadRequest)
			return
		}

		temp, ok := weatherService.GetTemperature(city)
		if !ok {
			weatherService.AddCity(city)
			temp, _ = weatherService.GetTemperature(city)
		}

		fmt.Fprintf(w, `{"city": "%s", "temperature": %d}`, city, temp)
	})

	if err := http.ListenAndServe(":3333", nil); err != nil {
		panic(err)
	}
}

```


# The thundering herd problem или эффект «стаи собак»

Эффект "стаи собак" ([The thundering herd problem](https://www.google.com/search?q=The+thundering+herd+problem&sourceid=chrome&ie=UTF-8&mstk=AUtExfCopCBA77TUl2wHhvSFoZJsA1Ea8-OA0ZDouTCQRn7NJCigOCRiHTXm7DEw-MVWXuuZRIG4FXQ8-qecCoJxZBIE1HoGv-r6daBv3MWO3gB65OmyFq7Zc5foo5Kmsjv11dI77j4Jr39-jeKFrLCoK7OZKRZGbyknkkgodZHGHyDGk4z72TQ9CflkTEnSG4Jb2lZyguRMFeTTFnbpiw4V4MSgWRQnFJDqv5KqU6NlGemjgo8t1AV33rB9BY-mCGmDI1u0O2YDZV_y7pqEZqhUxwtK&csui=3&ved=2ahUKEwjUj8qc55yRAxVh9LsIHcFtCrAQgK4QegQIAhAB)) — это ==резкое увеличение нагрузки на систему, когда сотни или тысячи запросов одновременно обращаются к одному и тому же ресурсу, например, при истечении срока действия кэша==. Это приводит к тому, что все экземпляры приложения не находят нужный ключ в кэше и одновременно обращаются к базе данных или мастер-системе, создавая пиковую нагрузку. 

- **Когда возникает:** Обычно проявляется при истечении срока действия кэша ([TTL](https://www.google.com/search?q=TTL&sourceid=chrome&ie=UTF-8&mstk=AUtExfCopCBA77TUl2wHhvSFoZJsA1Ea8-OA0ZDouTCQRn7NJCigOCRiHTXm7DEw-MVWXuuZRIG4FXQ8-qecCoJxZBIE1HoGv-r6daBv3MWO3gB65OmyFq7Zc5foo5Kmsjv11dI77j4Jr39-jeKFrLCoK7OZKRZGbyknkkgodZHGHyDGk4z72TQ9CflkTEnSG4Jb2lZyguRMFeTTFnbpiw4V4MSgWRQnFJDqv5KqU6NlGemjgo8t1AV33rB9BY-mCGmDI1u0O2YDZV_y7pqEZqhUxwtK&csui=3&ved=2ahUKEwjUj8qc55yRAxVh9LsIHcFtCrAQgK4QegQIAxAB)) для популярного товара во время распродажи, как в случае с "Черной пятницей".
- **Как проявляется:**
    - Множество экземпляров приложения одновременно обнаруживают, что ключ в кэше отсутствует (истек срок действия).
    - Все эти запросы отправляются к мастер-системе, которая может не выдержать такую одновременную нагрузку.
    - После получения ответа от мастер-системы, все экземпляры пытаются записать новый ключ в кэш, что также приводит к пиковой нагрузке.
- **Последствия:** Система может стать недоступной или работать крайне медленно из-за резкого скачка нагрузки.