https://medium.com/pickme-engineering-blog/singleflight-in-go-a-clean-solution-to-cache-stampede-02acaf5818e3

![](https://miro.medium.com/v2/resize:fit:1005/1*6AVWjk7MBbHuhHNlDrm4EQ.jpeg)

The concept of **cache stampede** is a well-known challenge in distributed systems, especially under high load. When many clients simultaneously try to access an expired or missing cache entry, they can overwhelm the backend by making redundant requests. To address this issue efficiently in Go applications, the **singleflight** package, provided by the Go team in the `golang.org/x/sync` module, offers a clean and effective solution by deduplicating concurrent calls for the same resource.

## Cache Stampede

A cache stampede occurs when a cached item expires or is evicted, and multiple clients simultaneously request that same item. Since the data is no longer available in the cache, all of those requests fall back to the backend or database, causing a sudden surge in load. In extreme cases, this can lead to degraded performance or even total system outages.

## A Real-World Scenario

In a production environment, a service running inside a Kubernetes pod used Redis as its distributed caching layer. One of its endpoints, `/template-details`, fetched template information based on a template ID provided in each incoming request. Since these templates were updated frequently, on average every 30 seconds, caching played a vital role in reducing load on the underlying MySQL database.

However, during a high-traffic period, a network partition occurred between the pod and the Redis instance. As a result, every incoming request led to a cache miss. The service began querying the MySQL database for each request, causing a sudden and significant spike in database traffic. Because the database was a single point of failure, it quickly became overwhelmed and temporarily went down, which caused the entire system to become unavailable.

## The Solution — singleflight

To prevent this kind of cascading failure, Go provides the **singleflight** package through the `golang.org/x/sync` module. It ensures that concurrent requests for the same resource are deduplicated. Only one actual call is made to the backend, while the remaining concurrent calls wait and reuse the same result.

This simple yet powerful pattern can drastically reduce backend load and prevent outages caused by cache stampedes.

Of course, like any solution, singleflight comes with its own trade-offs and considerations.

In the remainder of this article, we’ll explore how singleflight works, walk through a practical implementation, and examine the pros and cons of using it in real-world Go applications.

## Practical Example

To validate the behavior of singleflight, we built a simple POC with three main components:

- A mock cache simulating Redis
- A mock repository simulating a database
- A business layer that applies singleflight to prevent redundant database calls

Below is the complete code with explanations for each part.

## Mock Cache

This in-memory mock cache supports basic Set, Get, and Delete operations. The `Get` method simulates a cache miss if the key has expired or doesn’t exist.

package distributed_cache  
  
import (  
    "context"  
    "fmt"  
    "time"  
)  
  
type Cache interface {  
    Set(ctx context.Context, key, value string, ttl time.Duration) error  
    Get(ctx context.Context, key string) (string, error)  
    Delete(ctx context.Context, key string)  
}  
  
type MockCache struct {  
    data map[string]mockEntry  
}  
  
type mockEntry struct {  
    value     string  
    expiresAt time.Time  
}  
  
func NewMockCache() *MockCache {  
    return &MockCache{data: make(map[string]mockEntry)}  
}  
  
func (m *MockCache) Set(ctx context.Context, key string, value string, ttl time.Duration) error {  
    m.data[key] = mockEntry{  
       value:     value,  
       expiresAt: time.Now().Add(ttl),  
    }  
    return nil  
}  
  
func (m *MockCache) Get(ctx context.Context, key string) (string, error) {  
    entry, found := m.data[key]  
    if !found || time.Now().After(entry.expiresAt) {  
       return "", fmt.Errorf("key not found or expired")  
    }  
    return entry.value, nil  
}  
  
func (m *MockCache) Delete(ctx context.Context, key string) {  
    delete(m.data, key)  
}

## Mock Repository

The following repository simulates a database. Each time the method `GetTemplateByID` is called, it increments a counter to track how many times the database is actually hit.

package database  
  
import (  
    "fmt"  
    "sync/atomic"  
)  
  
type Template struct {  
    Key  string  
    Name string  
}  
  
var counter int64  
  
type TemplateRepository interface {  
    GetTemplateByID(key string) (*Template, error)  
}  
  
type MockTemplateRepository struct{}  
  
func NewMockTemplateRepository() TemplateRepository {  
    return &MockTemplateRepository{}  
}  
  
func (m *MockTemplateRepository) GetTemplateByID(key string) (*Template, error) {  
    atomic.AddInt64(&counter, 1)  
    fmt.Println("database call hit: ", atomic.LoadInt64(&counter))  
    mockData := map[string]*Template{  
       "key1": {Key: "key1", Name: "test-1"},  
       "key2": {Key: "key1", Name: "test-2"},  
    }  
    if template, exists := mockData[key]; exists {  
       return template, nil  
    }  
    return nil, fmt.Errorf("template not found")  
}

## Business Logic

In this part, we apply singleflight to ensure that only less database call is made for a given key when the cache is missed.

package usecase  
  
import (  
    "context"  
    "fmt"  
    "github.com/singleflight-example/database"  
    "github.com/singleflight-example/distributed_cache"  
    "golang.org/x/sync/singleflight"  
)  
  
type templateObj struct {  
    templateRepo      database.TemplateRepository  
    cache             distributed_cache.Cache  
    singleFlightGroup *singleflight.Group  
}  
  
type Template interface {  
    GetTemplateNameById(ctx context.Context, key string) string  
}  
  
func NewTemplate(templateRepo database.TemplateRepository,  
    cache distributed_cache.Cache) Template {  
    return &templateObj{  
       templateRepo:      templateRepo,  
       cache:             cache,  
       singleFlightGroup: &singleflight.Group{},  
    }  
}  
  
func (t templateObj) GetTemplateNameById(ctx context.Context,  
    key string) string {  
    name, err := t.cache.Get(ctx, key)  
    if err != nil {  
       fmt.Println("error in getting template name in cache, err ", err)  
  
       result, err, _ := t.singleFlightGroup.Do(key, func() (interface{}, error) {  
          tmpl, err := t.templateRepo.GetTemplateByID(key)  
          if err != nil {  
             fmt.Println("error in getting template name in database: ", err)  
             return tmpl, err  
          }  
          //err = t.cache.Set(ctx, key, tmpl.Name, 1)  
          //if err != nil {  
          // return nil, err  
          //}  
          return tmpl.Name, nil  
       })  
       if err != nil {  
          return ""  
       }  
  
       return (result).(string)  
  
    }  
    return name  
}

## main.go

The following `main.go` initializes the mock cache, database, and business logic layer. It then simulates 100 concurrent requests for the same key using goroutines. This setup mimics a cache stampede scenario (as we don’t set any key value to mock cache) and shows how singleflight effectively reduces backend pressure.

package main  
  
import (  
    "context"  
    "github.com/singleflight-example/database"  
    "github.com/singleflight-example/distributed_cache"  
    "github.com/singleflight-example/usecase"  
    "time"  
)  
  
func main() {  
    ctx := context.Background()  
    // database initialization  
    templateRepo := database.NewMockTemplateRepository()  
  
    // redis initialization  
    cache := distributed_cache.NewMockCache()  
  
    // use case initialization  
    templateUsecase := usecase.NewTemplate(templateRepo, cache)  
  
    for i := 0; i < 100; i++ {  
       go func(i int) {  
          _ = templateUsecase.GetTemplateNameById(ctx, "key1")  
  
       }(i)  
    }  
    time.Sleep(5 * time.Second)  
}

## Expected Output

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:1094/1*CfqzNMbKqJahGhBU7goxGw.png)

The output shows that the database call counter is much lower than the number of goroutines, proving that the load to the backend/database is decreased with the singleflight package.

## Get Dilan Dashintha’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Remember me for faster sign in

You can find the complete source code for this example on [GitHub](https://github.com/dylan96dashintha/singleflight-example)

## How singleflight Works

The `singleflight` package is built around a type called `Group`. This group ensures that only one request is made for a given key, even if multiple goroutines ask for it simultaneously.

**How it works:**

1. **First Request:** When the first request for a specific key arrives, singleflight starts the actual function call (e.g., fetching from the database).
2. **While It’s Running:** If more requests come in for the same key while the first one is still processing, they don’t trigger new calls. Instead, they wait.
3. **Sharing the Result:** Once the first function finishes, its result is shared with all waiting requests.
4. **No Duplicates:** The function is only called once, no matter how many requests arrive at the same time. This prevents duplicate work.

## Advantages of using Singleflight

- **Less Repeated Work:** Stops the system from doing the same task multiple times concurrently.
- **Better Performance:** Avoiding duplicate work allows faster response times and smoother handling of many users.
- **Stops Thundering Herd:** Prevents many requests from hitting the backend all at once, avoiding overload.

## Disadvantages of using Singleflight

- **Uses More Memory:** Tracking ongoing requests consumes extra memory, especially with many simultaneous requests.
- **Small Delays:** Some requests wait for the first one to finish, potentially adding small delays.
- **Not for Every Situation:** Works best when many requests ask for the same data. If data changes frequently or requests vary widely, it might not help much.

## Summary

Cache stampedes can severely impact distributed systems by overwhelming backends with redundant requests when cache entries expire. The Go `singleflight` package offers a clean and effective way to prevent these stampedes by ensuring that only one request for the same key is in-flight at any time. Other requests wait for the first to complete and reuse its result. This pattern reduces backend load, improves performance, and prevents system outages under high concurrency. While singleflight has some trade-offs like increased memory use and minor request delays, it is a valuable tool for Go developers building scalable, resilient caching layers.