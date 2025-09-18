https://yourbasic.org/golang/named-return-values-parameters/

In Go return parameters may be named and used as regular variables. When the function returns, they are used as return values.

```go
func f() (i int, s string) {
	i = 17
	s = "abc"
	return // same as return i, s
}
```

Named return parameters are initialized to their [zero values](https://yourbasic.org/golang/default-zero-value/).

The names are not mandatory but can make for good documentation. Correctly used, named return parameters can also help clarify and clean up the code.

## Example

This version of [`io.ReadFull`](https://golang.org/pkg/io/#ReadFull), taken from [Effective Go](https://golang.org/doc/effective_go.html#named-results), uses them to good effect.

```go
// ReadFull reads exactly len(buf) bytes from r into buf. It returns
// the number of bytes copied and an error if fewer bytes were read.
func ReadFull(r Reader, buf []byte) (n int, err error) {
	for len(buf) > 0 && err == nil {
		var nr int
		nr, err = r.Read(buf)
		n += nr
		buf = buf[nr:]
	}
	return
}
```

The code is both simpler and clearer, with named return values that are properly initialized and tied to a plain return.