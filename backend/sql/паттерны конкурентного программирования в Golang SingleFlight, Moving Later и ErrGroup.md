Работать с различными хранилищами данных — это ежедневная задача Go-разработчиков. Но что делать, если перед тобой не просто база данных, а сложная система? Например, база данных с репликацией, пошардированная база данных, либо база данных совместно используемая с кэшем.  
  
К счастью, в Go есть паттерны, которые делают такие задачи проще и эффективнее. В этой статье мы разберем 3 популярных паттерна конкурентного программирования, которые помогут справляться со сложностями:  

- Single Flight для работы с кэшем и базой данных;
- Moving Later для распределенных запросов в кластер баз данных с синхронными репликами;
- Err Group для работы с кластером базы данных с несколькими шардами.

Еще расскажу, как часто разработчики сталкиваются с такими задачами на собеседованиях и в реальной работе. Данные основаны на опросе 395 специалистов. А в конце статьи тебя ждет подборка видео и статей, чтобы углубить знания о Concurrency в Go

## 1. Single Flight: работа с кэшем и базой данных

Представь ситуацию: в MySQL хранятся пользовательские данные, а Redis используется для кэширования данных популярных пользователей. Типичный кейс для разработки.  
  
Для взаимодействия с кэшем часто выбирают подход ленивого кэширования (lazy caching), также известный как cache aside. Именно такой мы и рассмотрим ниже.

![](https://static.tildacdn.com/tild3631-6661-4434-a662-333930653733/Frame_282.svg)

###  Как это работает?

1. Ищем данные в кэше;
2. Если в кэше их нет, запрашиваем их из базы данных;
3. Добавляем полученные данные в кэш;
4. Отдаем результат пользователю.

  
Учебный пример реализации без использования отдельных слоев и прочего выглядит так:

```go
type Cache interface {
	Get(ctx context.Context, key string) (any, error)
	Set(ctx context.Context, key string, value any) error
}

type Database interface {
	Query(ctx context.Context, query string, args ...string) (any, error)
}

func GetUserBalance(ctx context.Context, userID string) (any, error) {
	value, err := cache.Get(ctx, userID)
	if err == nil {
		return value, nil
	}

	const query = "SELECT balance FROM users WHERE user_id = ?"
	value, err = database.Query(ctx, query, userID)
	if err != nil {
		return nil, err
	}

	_ = cache.Set(ctx, userID, value)
	return value, err
}
```

На первый взгляд всё кажется простым и удобным. Однако в условиях больших нагрузок этот код может привести к серьёзным проблемам.

### Проблема больших нагрузок

Представь, что система обрабатывает тысячи запросов в секунду (RPS). Если «горячий» ключ в кэше становится недействительным, тысячи пользователей могут одновременно попытаться его получить. Данные не найдутся в кэше, и все эти запросы направятся в базу данных.  
  
Что в результате?  

1. перегрузка базы данных;
2. возможный сбой или отказ базы данных.

  
Этот сценарий известен как [Thundering Herd Problem](https://en.wikipedia.org/wiki/Thundering_herd_problem). Чтобы справиться с этой проблемой, используется паттерн Single Flight.

### Как работает Single Flight?

Single Flight решает проблему конкурентных запросов так:  

1. Когда несколько горутин запрашивают один и тот же ключ, паттерн пропускает только одну из них с запросом в базу данных.
2. Остальные горутины ожидают, пока первая получит ответ из базы данных и затем получают данные.

### Как это реализовать?

В Go есть готовая библиотека для этого паттерна — [singleflight](https://pkg.go.dev/golang.org/x/sync/singleflight). Но для лучшего понимания мы напишем свою примитивную версию с нуля. Это поможет разобраться в деталях работы.

```go
type call struct {
	err   error
	value any
	done  chan struct{}
}

type SingleFlight struct {
	mutex sync.Mutex
	calls map[string]*call
}

func NewSingleFlight() *SingleFlight {
	return &SingleFlight{
		calls: make(map[string]*call),
	}
}

func (s *SingleFlight) Do(ctx context.Context, key string, action func(context.Context) (any, error)) (any, error) {
    s.mutex.Lock()
	if call, found := s.calls[key]; found {
		s.mutex.Unlock()
		return s.wait(ctx, call)
	}

	call := &call{
		done: make(chan struct{}),
	}

	s.calls[key] = call
	s.mutex.Unlock()

	go func() {
		defer func() {
			if v := recover(); v != nil {
				call.err = errors.New("error from single flight")
			}

			close(call.done)

			s.mutex.Lock()
			delete(s.calls, key)
			s.mutex.Unlock()
		}()

		call.value, call.err = action(ctx)
	}()

	return s.wait(ctx, call)
}

func (s *SingleFlight) wait(ctx context.Context, call *call) (any, error) {
	select {
	case <-ctx.Done():
		return nil, ctx.Err()
	case <-call.done:
		return call.value, call.err
	}
}
```

Теперь наш код для работы с кэшем и базой данных будет выглядеть так:

```go
type Cache interface {
	Get(ctx context.Context, key string) (any, error)
	Set(ctx context.Context, key string, value any) error
}

type Database interface {
	Query(ctx context.Context, query string, args ...string) (any, error)
}

func GetUserBalance(ctx context.Context, userID string) (any, error) {
	value, err := cache.Get(ctx, userID)
	if err == nil {
		return value, nil
	}

	const query = "SELECT balance FROM users WHERE user_id = ?"
	return singleFlight.Do(ctx, userID, func(ctx context.Context) (any, error) {
		value, err = database.Query(ctx, query, userID)
		if err != nil {
			return nil, err
		}

		_ = cache.Set(ctx, userID, value)
		return value, err
	})
}
```

Тем не менее, у этого примера все равно еще есть несколько проблем, с которыми можно потенциально столкнуться в реальных условиях:

### 1. Медленное выполнение первой горутины

Первая горутина, запрашивающая данные из базы данных, может выполняться намного дольше, чем ожидалось. Это может быть связано с:  

- медленным сетевым соединением;
- временной перегрузкой сервера;
- другими внешними факторами.

  
Остальные горутины будут вынуждены ждать завершения первой горутины, что увеличивает общее время выполнения.

Решение: можно использовать таймауты. Таймаут позволяет задать максимальное время выполнения первой горутины. Если она не успела завершиться, её выполнение прерывается и пропускается в хранилище за данными какая-нибудь другая горутина.

### 2. Устаревшие данные

Данные, полученные из базы данных, могут часто меняться. Это приводит к тому, что к моменту завершения запроса первой горутины результат устаревает.

Решение: сделать ключ недействительным и запустить новый запрос из другой горутины для обновления данных.

[](https://balun.courses/courses/concurrency/patterns_full)

## 2. Moving Later для работы с синхронными репликами в кластере

Представим, что у нас есть кластер базы данных PostgreSQL с синхронной репликацией, где один узел является ведущим (master), а остальные — репликами (slave). Кластер состоит из трех реплик, в нем хранятся данные о заказах, и наша цель — находить заказы по их идентификаторам за минимально-возможное время.  
  
Поскольку у нас в кластере синхронная репликация, можем выполнить такой алгоритм работы с кластером:  
  
1) Параллельные запросы ко всем репликам. Мы можем отправить запросы ко всем репликам одновременно. Это позволяет получить ответ как можно быстрее от той реплики, которая отреагирует первой.

![](https://static.tildacdn.com/tild3136-6238-4533-b862-643630336530/1.svg)

2) Игнорирование лишних ответов. Как только одна из реплик возвращает ответ, мы сразу же отдаём его пользователю. Все остальные ответы от реплик игнорируются, так как «данные во всех репликах одинаковые».

Потенциальная проблема — параллельные запросы создают дополнительную нагрузку на кластер.

В условиях высокой нагрузки или ограниченных ресурсов это может привести к перегрузке кластера, поэтому данный подход стоит применять с осторожностью и только в случаях, где критично минимальное время ответа.

![](https://static.tildacdn.com/tild3339-3633-4232-b862-653738393939/2.svg)

### Реализация паттерна Moving Later

Для реализации этого подхода используем паттерн Moving Later. Для демонстрации паттерна используется учебный пример с простым интерфейсом базы данных:

```go
type Database interface {
	Query(query string) string // simple interface for example
}

func DistributedQuery(replicas []Database, query string) string {
	responseCh := make(chan string, 1)
	for _, replica := range replicas {
		go func() {
			select {
			case responseCh <- replica.Query(query):
			default:
				return
			}
		}()
	}

	return <-responseCh
}

func main() {
	replicas := []*PgSQLDatabase{
		NewPgSQLDatabase("127.0.0.1:5432"),
		NewPgSQLDatabase("127.0.0.2:5432"),
		NewPgSQLDatabase("127.0.0.3:5432"),
	}

	response := DistributedQuery(replicas, "query to pgsql...")
	_ = response
}
```

### Зачем нужен буферизированный канал?

Если запись в канал не может пройти сразу, будет выбрана ветка в select по умолчанию. Поэтому выполнение записи без блокировки гарантирует, что ни одна из запущенных в цикле горутин не останется бесконечно висеть и канал можно не закрывать.  
  
Однако, если запись пройдет до того, как выполнение функции DistributedQuery добралось до чтения из канала (крайне маловероятная ситуация, но тем не менее), отправка может завершиться неудачей, потому что никто еще не готов, и для всех горутин в селекте будет выбрана ветка по умолчанию.

Решение: буферизированный канал гарантирует, что запись в него всегда будет успешной, и первое значение будет получено независимо от порядка выполнения.

- Вопрос:
    
    Нужно ли закрывать канал в данном примере?
    
    Ответ:
    
    Нет, закрывать канал необязательно.
    

[](https://balun.courses/courses/concurrency/patterns_full)

## 3. Err Group для запросов в кластер с несколькими шардами

Предположим, что у нас есть кластер базы данных ClickHouse, состоящий из нескольких шардов. Также допустим, что у ClickHouse старая версия, которая не поддерживает Distributed Table Engine. В каждом шарде хранятся данные о продажах разных магазинов крупной розничной сети.  
  
Задача — написать распределённый запрос, который будет обращаться ко всем шардам и собирать общую выручку за последнюю неделю со всех магазинов.

[](https://balun.courses/courses/concurrency/patterns_full)

### Зачем нам дожидаться всех шардов?

В отличие от паттерна Moving Later, в этом случае нам нужно дождаться ответа от всех шардов. Это важно, потому что каждый шард хранит только свою часть данных, и для получения корректной общей выручки нужно учесть все данные из разных шардов.

![](https://static.tildacdn.com/tild3135-6235-4634-b438-613832343036/3.svg)

Ниже представлена простая учебная реализация с базовым интерфейсом базы данных без дополнительных слоев:

```go
type Database interface {
	Query(query string) (string, error) // simple interface for example
}

func DistributedQuery(shards []Database, query string) []string {
	var wg sync.WaitGroup
	wg.Add(len(shards))

	responseCh := make(chan string)
	for _, shard := range shards {
		go func() {
			defer wg.Done()
			response, _ := shard.Query(query)
			responseCh <- response
		}()
	}

	go func() {
		wg.Wait()
		close(responseCh)
	}()

	responses := make([]string, 0, len(shards))
	for response := range responseCh {
		responses = append(responses, response)
	}

	return responses
}

func main() {
	shards := []*ClickHouseDatabase{
		NewClickHouseDatabase("127.0.0.1:5432"),
		NewClickHouseDatabase("127.0.0.2:5432"),
		NewClickHouseDatabase("127.0.0.3:5432"),
	}

	response := DistributedQuery(shards, "query to clickhouse...")
	_ = response
}
```

### Проблема отказа одного из шардов

Если мы не получим ответ от одного из шардов (например, из-за проблемы с сетью или отказа шарда), то нужно прервать все остальные запросы и вернуть ошибку пользователю

Это необходимо сделать, потому что ответ будет неполным, и дальнейшее ожидание данных из других шардов не имеет смысла.

![](https://static.tildacdn.com/tild3831-6435-4337-b365-396464373564/4.svg)

### Решение с помощью паттерна Err Group

Для решения этой проблемы мы используем паттерн Err Group. Этот паттерн:  

1. дожидается выполнения всех запросов;
2. отменяет все остальные запросы, если один завершился ошибкой.

Таким образом, если один из шардов не отвечает, все остальные запросы также будут отменены, и мы получим корректную ошибку, не тратя лишнего времени

### Реализация

Реализация этого паттерна [уже существует](https://pkg.go.dev/golang.org/x/sync/errgroup), и её можно использовать. Однако в этой статье мы напишем свою примитивную версию с нуля. Это поможет разобраться в деталях работы.

```go
type Group struct {
	cancel func(error) // this is an anti-pattern, but ...
	wg     sync.WaitGroup

	errOnce sync.Once
	err     error
}

func NewErrGroup(ctx context.Context) (*Group, context.Context) {
	ctx, cancel := context.WithCancelCause(ctx)
	return &Group{cancel: cancel}, ctx
}

func (g *Group) Go(action func() error) {
	g.wg.Add(1)
	go func() {
		defer g.wg.Done()
		if err := action(); err != nil {
			g.errOnce.Do(func() {
				g.err = err
				g.cancel(g.err)
			})
		}
	}()
}

func (g *Group) Wait() error {
	g.wg.Wait()
	return g.err
}
```

Теперь код, использующий этот паттерн, будет выглядеть так:

```go
type Database interface {
	Query(query string) (string, error) // simple interface for example
}

func DistributedQuery(shards []Database, query string) ([]string, error) {
	var mutex sync.Mutex
	responses := make([]string, 0, len(shards))
	group, ctx := NewErrGroup(context.TODO()) // nice to have parent context

	for _, shard := range shards {
		group.Go(func() error {
			type result struct {
				response string
				err      error
			}

			resultCh := make(chan result, 1)
			go func() {
				response, err := shard.Query(query)
				resultCh <- result{response: response, err: err}
			}()

			select {
			case <-ctx.Done():
				return ctx.Err()
			case result := <-resultCh:
				if result.err != nil {
					return result.err
				}

				mutex.Lock()
				responses = append(responses, result.response)
				mutex.Unlock()

				return nil
			}
		})
	}

	if err := group.Wait(); err != nil {
		return nil, err
	} else {
		return responses, nil
	}
}

func main() {
	shards := []*ClickHouseDatabase{
		NewClickHouseDatabase("127.0.0.1:5432"),
		NewClickHouseDatabase("127.0.0.2:5432"),
		NewClickHouseDatabase("127.0.0.3:5432"),
	}

	response, err := DistributedQuery(shards, "query to clickhouse...")
	_ = response
	_ = err
}
```

- Вопрос:
    
    Можно ли вместо err group использовать context.WithCancel?
    
    Ответ:
    
    Да, но это потребует дополнительных усилий. Например, использования sync.WaitGroup для ожидания горутин и проброс ошибки из одной из них.
    

[](https://balun.courses/courses/concurrency/patterns_full)

##   как часто это встречается в работе и на собеседованиях

Недавно мы провели опрос, чтобы узнать, часто ли разработчики сталкиваются с подобными задачами на собеседованиях или в работе. В опросе участвовали 395 человек. Вот результаты:  
  

- Часто встречались такие задачи — 19%
- Не часто встречались такие задачи — 36%
- Никогда не встречались такие задачи — 45%

![](https://static.tildacdn.com/tild3562-3962-4539-a538-336362396165/Slide_16_9_-_2.svg)

[](https://balun.courses/courses/concurrency/patterns_full)

### Дополнительная литература

Если хочешь лучше разобраться в конкурентном и параллельном программировании, посмотри бесплатные видеокурсы на YouTube:  

- [Теория и практика многопоточной синхронизации](https://www.youtube.com/playlist?list=PL4_hYwCyhAvYTxm55RBm_HA5Bq5W1Nv-R)
- [Параллельное программирование](https://www.youtube.com/playlist?list=PLlb7e2G7aSpTBs1GPt-4UygYxK3bVSyZe)

  
Также рекомендую несколько полезных статей:  
  
Примитивы синхронизации в Go:  

- [Танцы с мьютексами в Go](https://habr.com/ru/articles/271789/)
- [Implementing reader-writer locks](https://eli.thegreenplace.net/2019/implementing-reader-writer-locks/)
- [Mutex and Lock Internals in Golang](https://blog.stackademic.com/mutex-internals-in-golang-1624749f35a6)
- [Reusable barriers in Golang](https://medium.com/golangspec/reusable-barriers-in-golang-156db1f75d0b)

  
Основные проблемы в конкурентном программировании:  

- [Race condition и Data Race](https://medium.com/german-gorelkin/race-8936927dba20)
- [Deadlocks, Livelocks и Starvation](https://medium.com/german-gorelkin/deadlocks-livelocks-starvation-ccd22d06f3ae)

  
Работа с каналами и контекстами:  

- [Анатомия каналов в Go](https://habr.com/ru/articles/490336/)
- [Безопасная работа с каналами в Go](https://medium.com/german-gorelkin/data-protected-by-confinement-ec6b7be401ea)
- [Разбираемся с пакетом Context в Golang](https://habr.com/ru/companies/nixys/articles/461723/)

  
Оптимизация и ошибки:  

- [Частые ошибки при разработке lockfree-алгоритмов и их решения](https://habr.com/ru/articles/174369/)
- [Синхронизация в потокобезопасных контейнерах](https://alexeykalina.github.io/technologies/synchronization.html)