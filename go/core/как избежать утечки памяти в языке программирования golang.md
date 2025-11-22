https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac

Содержание:

1. [Утечки памяти при работе со срезами](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic1)
2. [Утечки памяти при работе со словарями](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic2)
3. [Утечки памяти при работе со строками](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic3)
4. [Утечки памяти при работе с финализиторами](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic4)
5. [Утечки памяти при работе с каналами и примитивами синхронизации](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic5)
6. [Дополнительные материалы](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#materials)

Утечка памяти — это проблема, с которой сталкиваются разработчики, когда программа выделяет память, но не освобождает её, когда она больше не нужна. Со временем это приводит к нехватке доступной памяти и может вызвать сбои в работе программы.  
  
Принято считать, что утечки памяти характерны только для языков с ручным управлением памятью, таких как C и C++. Но что насчёт Go, который оснащён автоматическим сборщиком мусора? Может ли здесь возникнуть подобная проблема?  
  
Оказывается, может. Даже в Go, где за освобождение памяти отвечает сборщик мусора, утечки памяти всё же возможны. Например, если программа непреднамеренно  
сохраняет указатели на объекты, которые больше не используются, сборщик мусора не сможет освободить эти ресурсы. Таким образом, неправильное обращение с ресурсами в Go может привести к утечкам памяти.

### Причины утечек памяти в Go

В этой статье мы разберём несколько ситуаций, в которых могут возникать утечки памяти при работе:  

- Со срезами
- Со словарями
- Со строками
- С финализиторами
- С каналами и примитивами синхронизации


## Утечки памяти

[1. При работе со срезами](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic1)

[2. При работе со словарями](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic2)

[5. При работе с каналам и...](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic5)

[3. При работе со строками](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic3)

[4. При работе с финализато...](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#topic4)

[6. Допматериалы](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac#materials)

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## 1. Утечки памяти при работе со срезами

Иногда на практике приходится работать с большими объемами данных. Например, представьте, что в функцию передается срез данных размером в один гигабайт:

```go
func findSequence(data []byte) []byte {
	for i := 0; i < len(data)-1; i++ {
		if data[i] == 0xFF && data[i+1] == 0xEE {
			return data[i+2 : i+12]
		}
	}

	return nil
}

func processBigData() {
	var data []byte
	// let's imagine that data (1GB) was read from a file

	sequence := findSequence(data)
	_ = sequence // using of sequence later
}
```

В этом примере функция findSequence возвращает срез размером всего 10 байт, который ссылается на исходный массив data (1 GB).

![](https://static.tildacdn.com/tild6261-3466-4639-b331-643131313562/1.svg)

Несмотря на то что массив data больше не нужен, сборщик мусора не сможет его освободить, потому что новый срез sequence продолжает ссылаться на данные из исходного массива. Память, выделенная под массив, может быть освобождена только целиком, а не частично.

Результат: утечка памяти на 1 GB — 10 B. Память остаётся занятой до тех пор, пока существует хотя бы одна ссылка на неё.

Чтобы решить эту проблему, нужно перестать ссылаться на исходный срез. Достаточно скопировать всего 10 байт, чтобы сборщик мусора освободил большой участок памяти. Например, так:

```go
func findSequence(data []byte) []byte {
	for i := 0; i < len(data)-1; i++ {
		if data[i] == 0xFF && data[i+1] == 0xEE {
			copied := make([]byte, 10)
			copy(copied, data)
			return copied
		}
	}

	return nil
}

func processBigData() {
	var data []byte
	// let's imagine that data (1GB) was read from a file

	sequence := findSequence(data)
	_ = sequence // using of sequence later
}
```

### Утечки памяти при ссылке на одно значение среза

Утечки памяти в Go могут возникать даже тогда, когда мы с использованием указателя ссылаемся на один элемент большого среза. Рассмотрим следующий пример:

```go
func findElement(data []int, target int) *int {
	for idx := range data {
		if data[idx] == target {
			return &data[idx]
		}
	}

	return nil
}

func processBigData() {
	var data []int
	// let's imagine that data (1GB) was read from a file

	pointer := findElement(data, 100_000)
	_ = pointer // using of sequence later
}
```

В этом примере мы возвращаем указатель. Он будет ссылаться только на один элемент исходного гигабайтного среза. Но после выполнения функции findElement сам исходный срез уже не нужен.

![](https://static.tildacdn.com/tild3462-3233-4137-b861-653136383663/2.svg)

Результат: утечка памяти на 8GB - 8B.

Решение аналогично предыдущему примеру: перестаем ссылаться на исходный срез и копируем объект.

### Утечки памяти при работе с вложенными срезами

Рассмотрим ещё один интересный пример утечки памяти:

```go
type Data struct {
	values []byte
}

func getPartialData() []Data {
	data := make([]Data, 1000) // ~24KB
	for i := 0; i < len(data); i++ {
		data[i] = Data{
			values: make([]byte, 1<<20),
		}
	}

	return data[:2]
}
```

В этом примере я специально допускаю небольшую утечку памяти размером 24KB — 16B и использую часть среза. Это решение связано с тем, что срез имеет небольшой размер, и такая утечка в некоторых ситуациях может не влиять на работу приложения (такое происходит не всегда).  
  
Но тут есть важная деталь. Сам срез действительно небольшой, но каждый его элемент содержит указатель на другой срез. Этот срез занимает уже 1MB памяти. Поэтому итоговая утечка будет не 24KB — 16B, а 1GB + 24KB — 16B.

![](https://static.tildacdn.com/tild3036-3166-4531-a532-623634303363/3.svg)

Это происходит, потому что я использую небольшой срез, который занимает мало места. Но каждый элемент этого среза связан с большим срезом, представляющим участок памяти на 1MB. Поэтому в примере память для 1000 срезов по 1MB не освобождается.

Чтобы решить проблему, нужно:  

1. Либо перестать ссылаться на исходный срез и скопировать только 2 элемента;
2. Либо обнулить ненужные элементы массива. Если утечка памяти размером 24KB — 16B допустима, можно продолжать ссылаться на массив data, но важно обнулить элементы, которые больше не нужны:

```go
type Data struct {
	values []byte
}

func getPartialData() []Data {
	data := make([]Data, 1000) // ~24KB
	for i := 0; i < len(data); i++ {
		data[i] = Data{
			values: make([]byte, 1<<20),
		}
	}

    clear(data[2:])
	return data[:2]
}
```

### Условная утечка памяти при изменении размера среза

Этот пример иллюстрирует ситуацию, которая формально не является утечкой памяти, но может привести к неэффективному использованию ресурсов:

```go
func main() {
	size := 1 << 30
	data := make([]int, size)
	for i := 0; i < size; i++ {
		data[i] = i
	}

	// using big data

	data = data[:0]

	// further I will work only
	// with a small number of values
}
```

Что происходит:  
Создается массив размером 1 GB и срез data, ссылающийся на него.  

1. После обнуления длины среза (data = data[:0]) его размер становится 0, но емкость (capacity) остается прежней.
2. Несмотря на то что мы больше не используем весь массив, сборщик мусора не освобождает память, так как срез data продолжает ссылаться на исходный массив.

Когда мы работаем с большим срезом и устанавливаем его размер в ноль, делая его пустым, память в размере 1GB всё равно не будет очищена.

![](https://static.tildacdn.com/tild3831-3964-4162-b737-343266313765/4.svg)

Срез останется такого же размера, даже если я буду работать только с его частью. Вся эта память будет занята, но не будет использоваться.

Чтобы решить проблему, нужно:  

1. Либо присвоить срезу nil. Так сборщик мусора сможет освободить память;
2. Либо уменьшить размер среза. Для этого самостоятельно выдели нужный объем памяти и перенеси туда данные из старого среза, а старый участок памяти в будущем освободит сборщик мусора, потому что на него не будет никто ссылаться.

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## 2. Утечки памяти при работе со словарями

Иногда в процессе работы приходится добавлять в словарь большое количество элементов. Рассмотрим пример, который не является классическим случаем утечки памяти, но всё же может привести к излишнему потреблению памяти:

```go
func main() {
	data := make(map[int][128]byte, 1_000_000)
	for i := 0; i < 1_000_000; i++ {
		data[i] = [128]byte{}
	}

	// using big data

	for i := 0; i < 1_000_000; i++ {
		delete(data, i)
	}

	// further I will work only 
	// with a small number of keys
}
```

Если удалить все значения из словаря, память всё равно не освободится. На моей машине словарь будет занимать около 293 МB, даже если я потом буду использовать только несколько ключей в словаре.

![](https://static.tildacdn.com/tild3836-3134-4434-a563-626131343263/5.svg)

Так происходит из-за особенностей работы словарей в Go. Бакеты словарей, в которых хранятся элементы, не освобождаются, даже если они становятся пустыми.

### Чтобы решить проблему, нужно:

1. Либо присвоить словарю nil. Это позволит сборщику мусора освободить память, выделенную для словаря, включая бакеты.
2. Либо использовать указатели на массив вместо массива размером 128 байт. Это позволит избежать излишнего потребления памяти, так как в бакетах будут храниться только указатели на массивы, которые будут обнулены, а не сами массивы.

Занимательный момент: если размер ключа или значения в словаре больше 128 байт, то в бакете будет храниться указатель на значение, а не само значение.

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## 3. Утечки памяти при работе со строками

Большие строки в программировании встречаются довольно часто. Рассмотрим пример, в котором в функцию передаётся строка размером в один гигабайт:

```go
func findSequence(data string) string {
	for i := 0; i < len(data)-1; i++ {
		if data[i] == '\n' && data[i+1] == '\t' {
			return data[i+2 : i+12]
		}
	}

	return ""
}

func processBigData() {
	var data string
	// let's imagine that data (1GB) was read from a file

	sequence := findSequence(data)
	_ = sequence // using of sequence later
}
```

В примере возвращается подстрока, состоящая всего из 10 символов, которая ссылается на исходную гигабайтную строку. После выполнения функции findSequence исходная строка уже не нужна.

![](https://static.tildacdn.com/tild3036-3930-4963-b234-303434306337/1.svg)

Так как подстрока всё ещё ссылается на исходную строку, сборщик мусора не освободит память, пока эта ссылка существует.

Результат: утечка памяти размером 1GB - 10B.

Чтобы решить эту проблему, нужно перестать ссылаться на исходную строку и просто скопировать 10 байт. Например так: strings. Clone (data[i+2: i+12]).

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## 4. Утечки памяти при работе с финализаторами

Иногда при работе с Go нужно использовать CGO для аллокации памяти. Чтобы избежать утечек и не забыть освободить память, можно использовать финализаторы.  
  
Когда больше не останется указателей на созданный объект CString — сборщик мусора обнаружит его, вызовет финализатор и только на следующей итерации сборки мусора освободит память, как в примере ниже:

```go
type CString struct {
	cpointer *C.char
}

func Allocate() *CString {
	str := CString{cpointer: C.allocate()}
	runtime.SetFinalizer(str, func(ptr *CString) {
		C.free(ptr.cpointer)
	})

	return str
}
```

Также финализаторы помогают закрывать разные ресурсы. С их помощью можно автоматически закрывать файлы или сетевые соединения, чтобы ничего не забыть. Например, они применяются в стандартной библиотеке для работы с файлами:

```go
func newFile(fd int, name string, kind newFileKind, nonBlocking bool) *File {
	// other implementation of function

	runtime.SetFinalizer(f.file, (*file).close)
	return f
}
```

### Проблемы финализаторов: циклические ссылки

Несмотря на пользу, финализаторы имеют ограничения. Одна из основных проблем — невозможность корректно работать с циклическими ссылками. Рассмотрим пример:

```go
type Foo struct {
	bar *Bar
}

type Bar struct {
	foo *Foo
}

func main() {
	foo := &Foo{}
	bar := &Bar{}

	foo.bar = bar
	bar.foo = foo

	runtime.SetFinalizer(foo, func(ptr *Foo) {
		fmt.Println("finalizer called on addr", ptr, "value is", *ptr)
	})

	runtime.GC()
	time.Sleep(time.Second)
}
```

В этом примере между объектами foo и bar возникает циклическая ссылка: foo.bar указывает на bar, а bar.foo на foo. Сборщик мусора не сможет вызвать финализатор для foo, а память для объектов foo и bar не гарантируется, что будет освобождена. Это может привести к настоящей утечке памяти.

![](https://static.tildacdn.com/tild3334-3333-4237-b733-643632333235/6.svg)

Важно: Сборщик мусора в Go является трассирующим, поэтому он способен корректно обрабатывать циклические ссылки, если не используются финализаторы.

Чтобы решить эту проблему, избегайте циклических ссылок при использовании финализаторов.

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## 5. Утечки памяти при работе с каналами и примитивами синхронизации

Одной из частых причин утечек памяти в Go являются утечки горутин, особенно при неправильной работе с каналами. Рассмотрим пример, где происходит утечка при чтении из канала:

```go
func printMessages(messagesCh chan string) {
	for message := range messagesCh {
		fmt.Println(message)
	}
}

func processMessages(messages []string) {
	messagesCh := make(chan string)
	go printMessages(messagesCh)

	for _, message := range messages {
		messagesCh <- message
	}
}
```

Что происходит:  
1. Мы записываем сообщения из среза messages в последовательном порядке.  
2. Затем горутина, которую мы создали, блокируется.

Это происходит потому, что:  

- В канал больше никто не пишет.
- Канал никто не закрывает.

Результат:  

- В памяти остаются данные заблокированной горутины (например, её стек и метаинформация);
- Канал тоже занимает память. Сборщик мусора его не освободит, так как на канал ссылается заблокированная горутина.

Чтобы избежать этой проблемы, необходимо закрыть канал, когда запись в него завершена.

### Утечки при записи в канал

Утечки могут происходить не только при чтении из канала, но и при записи в канал. Рассмотрим пример:

```go
func doQuery(string) string {
	return "" // imitation of long remote query
}

func doDistributedQuery(queries []string) string {
	resultCh := make(chan string)
	for _, query := range queries {
		go func() {
			result := doQuery(query)
			resultCh <- result
		}()
	}

	return <-resultCh
}
```

Что происходит в этом примере?  

1. Мы создаём канал resultCh для передачи результата от одной из горутин.
2. Для каждого значения из queries создаём горутину, которая выполняет запрос и записывает результат в канал.
3. Однако в функции doDistributedQuery мы возвращаем только первое значение из канала (<-resultCh), то есть в этом примере только одна горутина сможет записать данные в канал.

  
Значение из канала будет прочитано один раз внутри функции. Все другие горутины, которые попытаются записать в канал после этого, останутся заблокированными навсегда. Это приведет к утечкам, так как заблокированные горутины продолжат ссылаться на канал, как и в примере с чтением.

Чтобы решить эту проблему, можно:  

- сделать канал буферизированным, установив буфер равным len(queries) - 1.
- либо писать в канал без блокировки, используя select, чтобы избежать блокировки:

```go
func doQuery(string) string {
	return "" // imitation of long remote query
}

func doDistributedQuery(queries []string) string {
	resultCh := make(chan string, 1)
	for _, query := range queries {
		go func() {
			result := doQuery(query)
			select {
			case resultCh <- result:
			default:
			}
		}()
	}

	return <-resultCh
}
```

При использовании второго решения, если запись в канал не может пройти сразу, будет выбрана ветка в select по умолчанию. Выполнение записи без блокировки гарантирует, что ни одна из запущенных в цикле горутин не останется бесконечно висеть. Канал можно не закрывать, так как он, как и другие объекты, будет очищен сборщиком мусора.  
  
При использовании select нам все равно нужен буфферизированный канал с размером не более единицы. Если запись пройдет до того, как функция doDistributedQuery начнет читать из канала, что маловероятно, отправка может не удастся. Это произойдет, потому что никто еще не готов, и для всех горутин в select будет выбрана ветка по умолчанию.  
  
Утечки данных горутин еще могут происходить при неправильном использовании контекстов и при работе с time. Ticker до версии Go 1.23. Однако внутри их используются каналы, поэтому мы не будем отдельно рассматривать эти случаи. Примеры утечек с time. Ticker и контекстами будут похожи на примеры утечек данных горутин при работе с каналами.  
  
Утечки данных горутин могут возникать не только при работе с каналами, но и при неправильном использовании примитивов синхронизации из пакета sync. Например, горутина или несколько горутин могут заблокироваться навсегда, если неправильно использовать один мьютекс:

```go
func process() {
	var mutex sync.Mutex
	go func() {
		mutex.Lock()
		mutex.Lock() // endless waiting
	}()
}
```

Или, например, так, используя несколько мьютексов:

```go
func validate(lhs, rhs *sync.Mutex) {
	lhs.Lock()
	rhs.Lock()
}

func process() {
	var mutex1 sync.Mutex
	var mutex2 sync.Mutex

	// potential deadlock - the order in
	// which mutexes are captured is violated

	go validate(&mutex1, &mutex2)
	go validate(&mutex2, &mutex1)
}
```

[](https://balun.courses/courses/deep_go/memory_leak?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)

## Дополнительные материалы

Чтобы глубже понять детали и нюансы Go, можешь изучить несколько полезных статей:  

- [О массивах и срезах в G](https://habr.com/ru/articles/739754/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)[o](https://habr.com/ru/articles/739754/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Нарезаем массивы правильно в Go](https://habr.com/ru/articles/597521/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Пакеты bytes и strings](https://habr.com/ru/articles/307554/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Мапы в Go — уровень Pro](https://habr.com/ru/companies/avito/articles/774618/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Hashmap по версии Golang — часть 1](https://habr.com/ru/articles/704796/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Hashmap по версии Golang — часть 2](https://habr.com/ru/articles/717724/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Анатомия каналов в Go](https://habr.com/ru/articles/490336/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)
- [Как устроены каналы в Go](https://habr.com/ru/articles/308070/?utm_medium=youtube&utm_campaign=vladimir_balun&clckid=9e6bd5ac)