```go
import (
	"fmt"
	"context"
	"sync"
)

func fanin(ctx context.Context, chans []chan int) chan int {
	out := make(chan int)

	go func() {
		wg := &sync.WaitGroup{}
		for _, ch := range chans {
			wg.Add(1)

			go func() {
				defer wg.Done()
				for {
					select {
					case <-ctx.Done():
						return
					case v, ok := <-ch:
						if !ok {
							return
						}

						select {
							case <-ctx.Done():
								return
							case out <- v:
							
						}
					}
				}
			}()
		}
		wg.Wait()
		close(out)
	}()

	return out
}

```