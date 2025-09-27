https://www.tiredsg.dev/blog/golang-sentinel-error/

Sentinel errors use specific values to denote an error. This usually equates to performing an equality check to determine the error. Dave Cheney wrote a great article back in 2016 explaining [how we should handle errors gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully "Dave Cheney's error handling"). He wrote about what sentinel errors are and why they should be avoided. One of the reasons are as follows:

> Using sentinel values is the least flexible error handling strategy, as the caller must compare the result to predeclared value using the equality operator. This presents a problem when you want to provide more context, as returning a different error would will break the equality check.  
> - [Dave Cheney](https://dave.cheney.net/ "Dave Cheney")

## Errors in Go 1.13

This is no longer true with the introduction of Go 1.13 errors. With Go 1.13 errors, we can now add context to sentinel errors by wrapping them without breaking the equality check, all while still keeping the sentinel errors immutable. You can even [wrap multiple errors now](https://www.tiredsg.dev/blog/wrapping-multiple-errors-golang/).

To create a constant sentinel error, we create a custom error type `SentinelError` based on type string. The value of the string will be the specific error. To satisfy the requirement of an `error` type, we are required to implement the `Error() string` function. To make the error immutable, we declare the sentinel error as a constant.

```go
type SentinelError string

func (err SentinelError) Error() string {
	return string(err)
}

const (
	ErrNotFound SentinelError = "ErrNotFound"
)
```

We could then use the constant directly as an error when we want to return the error.

```go
func CheckExist() error {
	return ErrNotFound
}
```

## Equality Check

With Golang 1.13 errors, we now use `Is()` to determine if any error in the error’s tree matches the target error. For a custom error to leverage on the same functionality, we can extend the custom error to support this by implementing the `Is()` function to perform the equality check.

```go
type CustomError struct {
	Code  int
	Error error
}

func (err CustomError) Is(target error) bool {
	return target.Error() == err.Error
}
```

With `SentinelError` the default behavior of comparing the constant string satisfies the requirement of equality check when `errors.Is()` is called.

> An error is considered to match a target if it is equal to that target  
> - [Golang errors package doc](https://pkg.go.dev/errors#Is "errors package doc")

```go
func GetObject() error {
	// Perform some operations to get object, but got error.
	if err := CheckExist(); err != nil {
		return fmt.Errorf("error when checking object existence. %w", err)
	}
}

func main() {
	err := GetObject()
	if errors.Is(err, ErrNotFound) {
		// Matches ErrNotFound and handle error accordingly.
	}
}
```

## Conclusion

Some may argue that using sentinel errors creates a dependency between two packages, or that sentinel errors may become part of your public API. This concern is valid when developing a public package intended to be used by a large number of developers. However, in most cases, we develop internal components or microservices APIs that already have some level of coupling. The existence of an API requires documentation that outlines specific rules to follow. With well-defined rules, using sentinel errors can be the most straightforward method for consumers of your API to handle errors effectively.