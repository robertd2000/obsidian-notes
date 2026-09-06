```go
package singleflight

import "sync"

// call хранит состояние одного in-flight вызова
type call struct {
	wg  sync.WaitGroup
	val any
	err error
}

// Group — группа singleflight-вызовов
type Group struct {
	mu    sync.Mutex
	calls map[string]*call
}

// Do выполняет fn только один раз для данного ключа.
// Все остальные вызовы с тем же ключом блокируются и получают тот же результат.
func (g *Group) Do(key string, fn func() (any, error)) (any, error, bool) {
	g.mu.Lock()

	if g.calls == nil {
		g.calls = make(map[string]*call)
	}

	// Если вызов уже летит — просто ждём его
	if c, ok := g.calls[key]; ok {
		g.mu.Unlock()
		c.wg.Wait()           // ← блокировка до завершения первого вызова
		return c.val, c.err, true
	}

	// Первый вызов — создаём запись
	c := &call{}
	c.wg.Add(1)
	g.calls[key] = c
	g.mu.Unlock()

	// Выполняем функцию
	c.val, c.err = fn()
	c.wg.Done()

	// Убираем из map, чтобы следующие вызовы запустили fn заново
	g.mu.Lock()
	delete(g.calls, key)
	g.mu.Unlock()

	return c.val, c.err, false
}
```