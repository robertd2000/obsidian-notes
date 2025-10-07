https://habr.com/ru/articles/858490/

На просторах интернета наткнулся на интересный код, который заставил задуматься и вникнуть глубже в устройство планировщика Go.

> Верно для go 1.22

Почему данный код всегда будет выводить одинаковый результат?

```go
  

func main() {

    runtime.GOMAXPROCS(1)

  

    var wg sync.WaitGroup

  

    wg.Add(5)

    for i := 0; i < 5; i++ {

        go func() {

            fmt.Println(i)

            wg.Done()

        }()

    }

  

    wg.Wait()

}

  

// 4 0 1 2 3
```

## Преамбула

Для ответа на этот вопрос предлагаю сначала углубиться в runtime.GOMAXPROCS и как он устроен. Уверен, что всем Golang разработчикам хорошо известна модель G-M-P, но приведут ссылку на [документ](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit?tab=t.0#heading=h.mmq8lm48qfcw) от разработчиков планировщика. Из всего документа интересует вот этот абзац

> When a new G is created or an existing G becomes runnable, it is pushed onto a list of runnable goroutines of current P. When P finishes executing G, it first tries to pop a G from own list of runnable goroutines; if the list is empty, P chooses a random victim (another P) and tries to steal a half of runnable goroutines from it.

Горутины помещаются в локальный FIFO процессора P. В рамках документа P‑processor не означает, что это как‑то ассоциировано с CPU, на мой взгляд можно подобрать синоним «Исполнитель», те это набор ресурсов, которые необходимы для выполнения G.

Но здесь ничего не сказано о том, как работает GOMAXPROCS и поэтому решил углубиться в исходники runtime. Для начала посмотрим, что из себя представляет этот вызов [runtime.GOMAXPROCS](https://cs.opensource.google/go/go/+/master:src/runtime/debug.go;l=16;drc=ab55465098a0cd33007684091b573717a6ea54cf?q=GOMAXPROCS&ss=go%2Fgo)

```go
// GOMAXPROCS sets the maximum number of CPUs that can be executing
// simultaneously and returns the previous setting. It defaults to
// the value of [runtime.NumCPU]. If n < 1, it does not change the current setting.
// This call will go away when the scheduler improves.
func GOMAXPROCS(n int) int {
	if GOARCH == "wasm" && n > 1 {
		n = 1 // WebAssembly has no threads yet, so only one CPU is possible.
	}
	lock(&sched.lock)
	ret := int(gomaxprocs)
	unlock(&sched.lock)
	if n <= 0 || n == ret {
		return ret
	}
	stw := stopTheWorldGC(stwGOMAXPROCS)
	// newprocs will be processed by startTheWorld
	newprocs = int32(n)
	startTheWorldGC(stw)
	return ret
}
```

Здесь происходит установка переменной newprocs, а в комментариях к коду указано, что данный метод устанавливает максимальное количество доступных CPU, что подтверждается исходным кодом. Именно установка переменной и не более того.

Далее по коду видно, что это приводит к остановке stopTheWorldGC с указанием причины stwGOMAXPROCS и далее запуском startTheWorldGC.

Теперь предлагаю посмотреть, что происходит дальше внутри [startTheWorld](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;l=1686;drc=ab55465098a0cd33007684091b573717a6ea54cf)

```go
// reason is the same STW reason passed to stopTheWorld. start is the start
// time returned by stopTheWorld.
//
// now is the current time; prefer to pass 0 to capture a fresh timestamp.
//
// stattTheWorldWithSema returns now.
func startTheWorldWithSema(now int64, w worldStop) int64 {
	assertWorldStopped()
	mp := acquirem() // disable preemption because it can be holding p in a local var
	if netpollinited() {
		list, delta := netpoll(0) // non-blocking
		injectglist(&list)
		netpollAdjustWaiters(delta)
	}
	lock(&sched.lock)
	procs := gomaxprocs
	if newprocs != 0 {
		procs = newprocs
		newprocs = 0
	}
	p1 := procresize(procs)
```

Меня интересуют больше строки начиная с 18 (что эквивалентно 1685 в [исходном коде](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;l=1685;drc=ab55465098a0cd33007684091b573717a6ea54cf)). В коде устанавливается переменная procs и далее происходит [pocresize(procs)](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;drc=ab55465098a0cd33007684091b573717a6ea54cf;l=5699). А внутри уже как раз происходит освобождение ранее занятых ресурсов и инициализация новых.

```go
// Change number of processors.
//
// sched.lock must be held, and the world must be stopped.
//
// gcworkbufs must not be being modified by either the GC or the write barrier
// code, so the GC must not be running if the number of Ps actually changes.
//
// Returns list of Ps with local work, they need to be scheduled by the caller.
func procresize(nprocs int32) *p
```

Таким образом GOMAXPROCS устанавливает количество задействованых P в runtime, которые участвуют в выполнение G и никак не влияют на кол-во доступных M (Thread). M могут добавляться и удаляться при необходимости. Подробнее об этих случаях можно почитать в официальной документации по планировщику.

## Копаем дальше

Для более детального понимания картины, решил запустить код в режиме отладки, чтобы посмотреть как он реально выполняется. Результат прилагаю на сринах

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a0c/0df/6c8/a0c0df6c865f0c7ea7b74f9bf17e1ba9.png)

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c9a/65f/809/c9a65f809c84744c242487c06c524980.png)

Видно, что все G добавлены в очередь и стоят на паузе и лишь последняя добавленная G встала на выполнение в P с назначенным потоком Thread X. Это уже интересно. Чтобы понять почему такое происходит, решил взглянуть как G добавляется на выполнение после вызова go func(){}. Предлагаю ознакомиться с [runtime/proc.go](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;l=4990)

```go
// Create a new g running fn.
// Put it on the queue of g's waiting to run.
// The compiler turns a go statement into a call to this.
func newproc(fn *funcval) {
	gp := getg()
	pc := sys.GetCallerPC()
	systemstack(func() {
		newg := newproc1(fn, gp, pc, false, waitReasonZero)
		pp := getg().m.p.ptr()
		runqput(pp, newg, true)
		if mainStarted {
			wakep()
		}
	})
}
```

Здесь go func трансформируется в вызов newproc(*fn), внутри которого создается newproc и далее кладется в локальную очередь runqput. Выглядит, что все должно выводиться из очереди, как описано в документации в порядке добавления.

Изучаю как работает [runqput](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;drc=ab55465098a0cd33007684091b573717a6ea54cf;l=6714).

```go
// runqput tries to put g on the local runnable queue.
// If next is false, runqput adds g to the tail of the runnable queue.
// If next is true, runqput puts g in the pp.runnext slot.
// If the run queue is full, runnext puts g on the global queue.
// Executed only by the owner P.
func runqput(pp *p, gp *g, next bool)
```

В этой функции интересует только строка [6730](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;l=6730;drc=ab55465098a0cd33007684091b573717a6ea54cf) где видно, что когда передается next, то происходит установка runnext внутри P. А в функции newproc, вызов как раз происходит с значением true. Кажется, что развязка близка. Можно узнать, что означает эта переменная [runnext](https://cs.opensource.google/go/go/+/master:src/runtime/runtime2.go;drc=ab55465098a0cd33007684091b573717a6ea54cf;l=659).

```go
// runnext, if non-nil, is a runnable G that was ready'd by
// the current G and should be run next instead of what's in
// runq if there's time remaining in the running G's time
// slice. It will inherit the time left in the current time
// slice. If a set of goroutines is locked in a
// communicate-and-wait pattern, this schedules that set as a
// unit and eliminates the (potentially large) scheduling
// latency that otherwise arises from adding the ready'd
// goroutines to the end of the run queue.
//
// Note that while other P's may atomically CAS this to zero,
// only the owner P can CAS it to a valid G.
runnext guintptr
```

Планировщик берет G на выполнение именно из этой переменной, а после уже смотрит локальную очередь, а при отсутствии работы, лезет в глобальную очередь. Бинго!

## Проверяем ИИ на присутствие интеллекта (Бонус)

Решил не останавливаться на достигнутом и проверить, справится ли ИИ с вопросом. Для проверки решил воспользоваться Claude 3.5 Sonnet через сервис [Cody](https://sourcegraph.com/cody/chat). Прежде чем задавать конкретный вопрос, решил добавить контекста, задав вопросы к ИИ про GOMAXPROCS, устройство планировщика Golang и другие наводящие вопросы. Получив довольно разумные ответы, задал конкретный вопрос.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a1b/7b9/02a/a1b7b902a2b9591b9c2e0fc54d826d07.png)

Предоставленный ответ оказался неверным (частично неверный, тк в Golang < 1.22 скорее всего так и было бы)

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/5e8/4bf/ef4/5e84bfef417bb2fcf3e8dddd28795285.png)

Пришлось сообщить, что ответ неверный и попросил исправиться. Но, ИИ решил, что он умный и начал доказывать, что его ответ верный. После чего, ему была предоставлена информация о том, что ответ будет 4 0 1 2 3. В итоге ИИ извинился и исправился, но предоставил какой-то абсурдный вывод.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a2f/b95/790/a2fb9579070c48e95279af664cae02f4.png)

Добавляю ИИ больше контекста и прошу его взглянуть на исходный код runtime.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ac8/e4c/878/ac8e4c8789dadeb65927501348d358ff.png)

В этот раз ИИ попытался подкрепить свой ответ ссылками на исходный код. Но вывод был неверный. В итоге, попросил обратить внимание на runqput и переменную runnext, что ожидаемо привело к верному ответу.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/448/313/4a2/4483134a2cf5f9601fa05701b5d7faa7.png)

Всем спасибо за внимание!