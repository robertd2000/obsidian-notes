
![[Pasted image 20250831182111.png]]

![[Pasted image 20250831182140.png]]


### Работа с гонками данных (data race) в Golang

**Гонки данных (Data race)** являются одними из самых распространенных и трудных для отладки типов ошибок в конкурентных системах. Гонка данных происходит, когда две программы одновременно обращаются к одной и той же переменной, и по крайней мере одно из обращений является записью.

Вот пример гонки данных, которая может привести к сбоям и повреждению памяти:

```go
func main() {
    c := make(chan bool)
    m := make(map[string]string)
    go func() {
        m["1"] = "a" // Первый конфликтующий доступ
        c <- true
    }()
    m["2"] = "b" // Второй конфликтующий доступ
    <-c
    for k, v := range m {
        fmt.Println(k, v)
    }
}
```

#### Использование детектора гонки данных

Чтобы помочь диагностировать такие ошибки, Go включает в себя встроенный **детектор гонки данных**. Чтобы использовать его, добавьте флаг -race к команде go:

```
$ go test -race mypkg    // тестирование пакета
$ go run -race mysrc.go  // запуск исходного файла
$ go build -race mycmd   // сборка команды
$ go install -race mypkg // установка пакета
```

#### Формат отчета

Когда детектор гонки обнаруживает гонку данных в программе, он печатает отчет. Отчет содержит трассировки стеков для конфликтующих доступов, а также стеки, в которых были созданы соответствующие go-процедуры (goroutines). Вот пример:

```
WARNING: DATA RACE
Read by goroutine 185:
  net.(*pollServer).AddFD()
      src/net/fd_unix.go:89 +0x398
  net.(*pollServer).WaitWrite()
      src/net/fd_unix.go:247 +0x45
  net.(*netFD).Write()
      src/net/fd_unix.go:540 +0x4d4
  net.(*conn).Write()
      src/net/net.go:129 +0x101
  net.func·060()
      src/net/timeout_test.go:603 +0xaf

Previous write by goroutine 184:
  net.setWriteDeadline()
      src/net/sockopt_posix.go:135 +0xdf
  net.setDeadline()
      src/net/sockopt_posix.go:144 +0x9c
  net.(*conn).SetDeadline()
      src/net/net.go:161 +0xe3
  net.func·061()
      src/net/timeout_test.go:616 +0x3ed

Goroutine 185 (running) created at:
  net.func·061()
      src/net/timeout_test.go:609 +0x288

Goroutine 184 (running) created at:
  net.TestProlongTimeout()
      src/net/timeout_test.go:618 +0x298
  testing.tRunner()
      src/testing/testing.go:301 +0xe8
```

#### Параметры

Переменная окружения GORACE устанавливает параметры детектора гонки. Формат такой:

```
GORACE="option1=val1 option2=val2"
```

Варианты:

```
log_path (по умолчанию stderr): 
детектор гонки записывает свой отчет 
в файл с именем log_path.pid. 
Специальные имена stdout и stderr приводят к записи отчетов 
в стандартный вывод 
и стандартную вывод ошибок соответственно.

exitcode (по умолчанию 66): 
состояние выхода, используемое 
при выходе после обнаруженной гонки.

strip_path_prefix (по умолчанию ""): 
удаляет этот префикс из всех путей к сообщаемым файлам, 
чтобы сделать отчеты более краткими.

history_size (по умолчанию 1): 
История доступа к памяти для каждой программы составляет 
32K * 2 ** history_size элементов. 
Увеличение этого значения позволяет избежать 
ошибки "не удалось восстановить стек" 
("failed to restore the stack")
в отчетах за счет увеличения использования памяти.

halt_on_error (по умолчанию 0): 
управляет выходом из программы 
после сообщения о первой гонке данных.
```

Пример:

```
$ GORACE="log_path=/tmp/race/report strip_path_prefix=/my/go/sources/" go test -race
```

#### Исключая тесты

При сборке с флагом -race команда go определяет дополнительную тег сборки race. Вы можете использовать тег, чтобы исключить некоторый код и тесты при запуске детектора гонки. Несколько примеров:

```go
// +build !race

package foo

// Тест содержит гонку данных. Заведено issue.
func TestFoo(t *testing.T) {
    // ...
}

// Тест падает под детектором гонки из-за таймаутов.
func TestBar(t *testing.T) {
    // ...
}

// Тест выполняется слишком долго под детектором гонки.
func TestBaz(t *testing.T) {
    // ...
}
```

#### Как пользоваться

Для начала запустите свои тесты, используя детектор гонки (go test -race). Детектор гонки находит только гонки, которые происходят во время выполнения, поэтому он не может находить гонки в путях кода, которые не выполняются. Если ваши тесты имеют неполный охват, вы можете найти больше гонок, запустив двоичный файл, созданный с -race, при реалистичной рабочей нагрузке.

### Типичные гонки данных

Вот некоторые типичные данные гонки. Все они могут быть обнаружены с помощью детектора гонки.

#### Гонка на счетчике цикла

```go
func main() {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < 5; i++ {
        go func() {
            // Не та 'i' которую вы ожидаете.
            fmt.Println(i) 
            wg.Done()
        }()
    }
    wg.Wait()
}
```

Переменная i в литерале функции - это та же переменная, которая используется циклом, поэтому чтение в goroutine мчится с шагом цикла. (Эта программа обычно печатает 55555, а не 01234.) Программу можно исправить, сделав копию переменной:

```go
func main() {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < 5; i++ {
        go func(j int) {
            // Считывает локальную копию счетчика цикла.
            fmt.Println(j) 
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

#### Случайно разделяемая переменная

```go
// ParallelWrite записывает данные в file1 и file2, 
// возвращает ошибки.
func ParallelWrite(data []byte) chan error {
    res := make(chan error, 2)
    f1, err := os.Create("file1")
    if err != nil {
        res <- err
    } else {
        go func() {
            // Эта err разделяемая с main goroutine,
            // поэтому выполнение записи вызывает 
            // гонку с выполнением записи ниже.
            _, err = f1.Write(data)
            res <- err
            f1.Close()
        }()
    }

    // Вторая конфликтующая запись в err.
    f2, err := os.Create("file2") 
    if err != nil {
        res <- err
    } else {
        go func() {
            _, err = f2.Write(data)
            res <- err
            f2.Close()
        }()
    }
    return res
}
```

Исправление заключается в том, чтобы вводить новые переменные в goroutine (обратите внимание на использование **`:=`** ):

```go
...
_, err := f1.Write(data)
...
_, err := f2.Write(data)
...
```

#### Незащищенная глобальная переменная

Если следующий код вызывается из нескольких программ, это приводит к гонкам карты service. Одновременное чтение и запись одной и той же карты небезопасны:

```go
var service map[string]net.Addr

func RegisterService(name string, addr net.Addr) {
    service[name] = addr
}

func LookupService(name string) net.Addr {
    return service[name]
}
```

Чтобы сделать код безопасным, защитите доступ с помощью мьютекса:

```go
var (
    service   map[string]net.Addr
    serviceMu sync.Mutex
)

func RegisterService(name string, addr net.Addr) {
    serviceMu.Lock()
    defer serviceMu.Unlock()
    service[name] = addr
}

func LookupService(name string) net.Addr {
    serviceMu.Lock()
    defer serviceMu.Unlock()
    return service[name]
}
```

#### Примитивная незащищенная переменная

Гонки данных могут происходить и с переменными примитивных типов (bool, int, int64 и т. д.), Как в этом примере:

```go
type Watchdog struct{ last int64 }

func (w *Watchdog) KeepAlive() {
    // Первый конфликтующий доступ.
    w.last = time.Now().UnixNano() 
}

func (w *Watchdog) Start() {
    go func() {
        for {
            time.Sleep(time.Second)
            // Второй конфликтующий доступ.
            if w.last < time.Now().Add(-10*time.Second).UnixNano() {
                fmt.Println("No keepalives for 10 seconds. Dying.")
                os.Exit(1)
            }
        }
    }()
}
```

Даже такие "невинные" гонки данных могут привести к трудным для отладки проблемам, вызванным неатомарностью доступа к памяти, вмешательством в оптимизацию компилятора или проблемами переупорядочения доступа к памяти процессора.

Типичным решением этой гонки является использование канала или мьютекса. Чтобы сохранить поведение без блокировки, можно также использовать пакет sync/atomic.

```go
type Watchdog struct{ last int64 }

func (w *Watchdog) KeepAlive() {
    atomic.StoreInt64(&w.last, time.Now().UnixNano())
}

func (w *Watchdog) Start() {
    go func() {
        for {
            time.Sleep(time.Second)
            if atomic.LoadInt64(&w.last) < time.Now().Add(-10*time.Second).UnixNano() {
                fmt.Println("No keepalives for 10 seconds. Dying.")
                os.Exit(1)
            }
        }
    }()
}
```

#### Поддерживаемые системы

Детектор гонки работает на linux/amd64, linux/ppc64le, linux/arm64, freebsd/amd64, netbsd/amd64, darwin/amd64 и windows/amd64.

#### Время выполнения

Стоимость обнаружения гонки зависит от программы, но для типичной программы использование памяти может увеличиться в 5-10 раз, а время выполнения - в 2-20 раз.