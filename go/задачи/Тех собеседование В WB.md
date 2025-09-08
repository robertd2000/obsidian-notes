https://www.youtube.com/watch?v=KJIchjXmXJ8
# 1

## Условие
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"sync"
	"time"
)

// Задача в контексте сервиса, в котором клиенты
// могут задавать список хостов для периодической проверки доступности.
//
// В любое время клиент может запросить последний известный статус по всем хостам.
// В любое время клиент может добавлять и удалять хосты.
//
// Список хостов со статусами хранится в глобальном map[string]bool (для простоты задания)
//
// Задание: реализовать систему проверку доступности (health check) HTTP-хостов
//
// * каждый 'interval' секунд проверяем текущий статус всех хостов
// * в случае ошибки создания или выполнения любого запроса сразу же
// отменить все остальные проверки и возвышать первую ошибку
// * рассчитывать HTTP-статус '200 OK' как "живой" (true),
// любой другой статус — как "мертвый" (false)
//

hosts := make(map[string]bool)

func checkHealth(ctx context.Context, host string) (bool, error) {
		req, err := http.NewRequestWithContext(ctx, "GET", host, nil)
	if err != nil {
		return false, err
	}
}
 
func main() {
}
```


## Solution 1

```go

package main

import (
	"context"
	"fmt"
	"net/http"
	"sync"
	"time"
)

// HealthChecker представляет сервис проверки доступности хостов
type HealthChecker struct {
	hosts      map[string]bool
	mu         sync.RWMutex
	interval   time.Duration
	client     *http.Client
	ctx        context.Context
	cancel     context.CancelFunc
	isRunning  bool
	workerPool chan struct{}
}

// NewHealthChecker создает новый экземпляр HealthChecker
func NewHealthChecker(interval time.Duration, maxWorkers int) *HealthChecker {
	ctx, cancel := context.WithCancel(context.Background())

	return &HealthChecker{
		hosts:      make(map[string]bool),
		interval:   interval,
		workerPool: make(chan struct{}, maxWorkers),
		client: &http.Client{
			Timeout: 10 * time.Second,
		},
		ctx:    ctx,
		cancel: cancel,
	}
}

// AddHost добавляет хост для мониторинга
func (hc *HealthChecker) AddHost(host string) {
	hc.mu.Lock()
	defer hc.mu.Unlock()

	// Добавляем схему, если отсутствует
	if len(host) >= 7 && host[:7] != "http://" && 
	   len(host) >= 8 && host[:8] != "https://" {
		host = "http://" + host
	}

	hc.hosts[host] = false // Изначально статус неизвестен
}

// RemoveHost удаляет хост из мониторинга
func (hc *HealthChecker) RemoveHost(host string) {
	hc.mu.Lock()
	defer hc.mu.Unlock()

	delete(hc.hosts, host)
}

// GetStatuses возвращает текущие статусы всех хостов
func (hc *HealthChecker) GetStatuses() map[string]bool {
	hc.mu.RLock()
	defer hc.mu.RUnlock()

	// Создаем копию для безопасного возврата
	statuses := make(map[string]bool, len(hc.hosts))
	for host, status := range hc.hosts {
		statuses[host] = status
	}

	return statuses
}

// checkHost проверяет доступность одного хоста с поддержкой отмены
func (hc *HealthChecker) checkHost(ctx context.Context, host string) (bool, error) {
	// Создаем запрос с контекстом для поддержки отмены
	req, err := http.NewRequestWithContext(ctx, "GET", host, nil)
	if err != nil {
		return false, fmt.Errorf("failed to create request for %s: %w", host, err)
	}

	resp, err := hc.client.Do(req)
	if err != nil {
		// Проверяем, была ли отмена контекста
		select {
		case <-ctx.Done():
			return false, context.Canceled
		default:
			return false, fmt.Errorf("failed to execute request for %s: %w", host, err)
		}
	}
	defer resp.Body.Close()

	return resp.StatusCode == http.StatusOK, nil
}

// checkAllHosts проверяет все хосты с worker pool и отменой при первой ошибке
func (hc *HealthChecker) checkAllHosts() error {
	ctx, cancel := context.WithCancel(hc.ctx)
	defer cancel()

	var wg sync.WaitGroup
	errCh := make(chan error, 1)
	results := make(chan struct {
		host   string
		status bool
		err    error
	}, len(hc.hosts))

	// Получаем список хостов для проверки
	hc.mu.RLock()
	hosts := make([]string, 0, len(hc.hosts))
	for host := range hc.hosts {
		hosts = append(hosts, host)
	}
	hc.mu.RUnlock()

	// Запускаем проверку каждого хоста в worker pool
	for _, host := range hosts {
		// Ожидаем свободного воркера
		hc.workerPool <- struct{}{}

		wg.Add(1)

		go func(h string) {
			defer wg.Done()
			defer func() { <-hc.workerPool }()

			select {
			case <-ctx.Done():
				// Контекст отменен, пропускаем проверку
				return
			default:
				status, err := hc.checkHost(ctx, h)
				
				select {
				case results <- struct {
					host   string
					status bool
					err    error
				}{h, status, err}:
				case <-ctx.Done():
				}

				if err != nil && err != context.Canceled {
					select {
					case errCh <- err:
						cancel() // Отменяем все другие проверки
					default:
					}
				}
			}
		}(host)
	}

	// Ждем завершения всех горутин
	go func() {
		wg.Wait()
		close(results)
		close(errCh)
	}()

	// Собираем результаты и обрабатываем ошибки
	hasError := false
	var firstError error

	// Обрабатываем результаты и ошибки
	for result := range results {
		if result.err != nil && result.err != context.Canceled && !hasError {
			hasError = true
			firstError = result.err
			cancel() // Отменяем оставшиеся проверки
		}
		
		if result.err == nil {
			// Обновляем статус только для успешных проверок
			hc.mu.Lock()
			hc.hosts[result.host] = result.status
			hc.mu.Unlock()
		}
	}

	return firstError
}

// Start запускает периодическую проверку хостов
func (hc *HealthChecker) Start() {
	if hc.isRunning {
		return
	}

	hc.isRunning = true

	// Выполняем первую проверку сразу
	go hc.checkAllHosts()

	// Запускаем периодическую проверку
	go func() {
		ticker := time.NewTicker(hc.interval)
		defer ticker.Stop()

		for {
			select {
			case <-ticker.C:
				if err := hc.checkAllHosts(); err != nil {
					fmt.Printf("Health check failed: %v\n", err)
				}
			case <-hc.ctx.Done():
				return
			}
		}
	}()
}

// Stop останавливает проверку хостов
func (hc *HealthChecker) Stop() {
	hc.cancel()
	hc.isRunning = false
}

// GetHosts возвращает список всех отслеживаемых хостов
func (hc *HealthChecker) GetHosts() []string {
	hc.mu.RLock()
	defer hc.mu.RUnlock()

	hosts := make([]string, 0, len(hc.hosts))
	for host := range hc.hosts {
		hosts = append(hosts, host)
	}

	return hosts
}

// Пример использования
func main() {
	// Создаем health checker с интервалом 30 секунд и 5 воркерами
	hc := NewHealthChecker(10*time.Second, 5)

	// Добавляем хосты для мониторинга
	hc.AddHost("google.com")
	hc.AddHost("github.com")
	hc.AddHost("httpbin.org/status/200")
	hc.AddHost("httpbin.org/status/404") // Этот вернет 404
	hc.AddHost("invalid-host-that-will-fail.com") // Этот вызовет ошибку

	// Запускаем проверку
	hc.Start()

	// Даем время на первую проверку
	time.Sleep(2 * time.Second)

	// Получаем текущие статусы
	statuses := hc.GetStatuses()
	fmt.Println("Current statuses:")
	for host, status := range statuses {
		fmt.Printf("%s: %v\n", host, status)
	}

	// Ждем немного и показываем обновленные статусы
	time.Sleep(12 * time.Second)

	statuses = hc.GetStatuses()
	fmt.Println("\nUpdated statuses:")
	for host, status := range statuses {
		fmt.Printf("%s: %v\n", host, status)
	}

	// Показываем список всех хостов
	fmt.Println("\nAll monitored hosts:")
	for _, host := range hc.GetHosts() {
		fmt.Println(host)
	}

	// Останавливаем проверку
	hc.Stop()
	fmt.Println("\nHealth checker stopped")
}

```

