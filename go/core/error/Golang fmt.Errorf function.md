https://www.zetcode.com/golang/fmt-errorf/

This tutorial explains how to use the `fmt.Errorf` function in Go. We'll cover error creation basics with practical examples of formatted errors.

The fmt.Errorf function creates error values with formatted messages. It works similarly to `fmt.Printf` but returns an error instead of printing output. This is useful for creating descriptive error messages.

In Go, `fmt.Errorf` is commonly used to wrap errors with additional context. It supports all the formatting verbs available in `fmt.Printf`. The function returns an error value that implements the error interface.

## Basic fmt.Errorf example

The simplest use of `fmt.Errorf` creates an error with a static message. This example demonstrates basic error creation.  
**Note:** The error message can include any format specifiers.

basic_error.go

```go

package main

import (
    "fmt"
    "log"
)

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Result:", result)
}
```


The function returns a formatted error when division by zero is attempted. The main function checks the error and logs it if present.

## Formatting error messages

`fmt.Errorf` supports all standard formatting verbs. This example shows how to include dynamic values in error messages.

formatted_error.go

```go

package main

import (
    "fmt"
    "os"
)

func checkFileSize(filename string, maxSize int64) error {
    info, err := os.Stat(filename)
    if err != nil {
        return err
    }
    
    if info.Size() > maxSize {
        return fmt.Errorf("file %s is too large (%d bytes > max %d bytes)",
            filename, info.Size(), maxSize)
    }
    return nil
}

func main() {
    err := checkFileSize("data.txt", 1024)
    if err != nil {
        fmt.Println("Error:", err)
    }
}

```

The error message includes the filename, actual size, and maximum size. This provides detailed context about what went wrong.

## Wrapping errors with fmt.Errorf

`fmt.Errorf` can wrap existing errors using the `%w` verb. This example shows how to add context while preserving the original error.

wrapping_error.go

```go

package main

import (
    "errors"
    "fmt"
    "os"
)

func readConfig(file string) ([]byte, error) {
    data, err := os.ReadFile(file)
    if err != nil {
        return nil, fmt.Errorf("failed to read config file %s: %w", file, err)
    }
    return data, nil
}

func main() {
    _, err := readConfig("missing.conf")
    if err != nil {
        fmt.Println("Error:", err)
        
        var pathErr *os.PathError
        if errors.As(err, &pathErr) {
            fmt.Println("Underlying path error:", pathErr)
        }
    }
}

```

The `%w` verb wraps the original error while adding context. The main function can still access the underlying error using `errors.As`.

## Error with multiple values

`fmt.Errorf` can format multiple values into an error message. This example shows a complex error with several dynamic values.

multi_value_error.go

```go

package main

import (
    "fmt"
    "time"
)

func scheduleEvent(name string, when time.Time) error {
    if when.Before(time.Now()) {
        return fmt.Errorf("cannot schedule %s at %v (in the past, now is %v)",
            name, when.Format(time.RFC3339), time.Now().Format(time.RFC3339))
    }
    return nil
}

func main() {
    pastTime := time.Now().Add(-1 * time.Hour)
    err := scheduleEvent("meeting", pastTime)
    if err != nil {
        fmt.Println("Error:", err)
    }
}

```

The error includes the event name, scheduled time, and current time. All values are properly formatted for clear error reporting.

## Custom error types with fmt.Errorf

`fmt.Errorf` can be used with custom error types. This example shows how to combine formatted messages with structured error data.

custom_error.go

```go

package main

import (
    "errors"
    "fmt"
)

type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation error on %s: %s", e.Field, e.Message)
}

func validateUser(name string, age int) error {
    if name == "" {
        return fmt.Errorf("user validation: %w",
            ValidationError{Field: "name", Message: "cannot be empty"})
    }
    if age < 0 {
        return fmt.Errorf("user validation: %w",
            ValidationError{Field: "age", Message: "must be positive"})
    }
    return nil
}

func main() {
    err := validateUser("", -1)
    if err != nil {
        fmt.Println("Error:", err)
        
        var valErr ValidationError
        if errors.As(err, &valErr) {
            fmt.Printf("Field %s failed: %s\n", valErr.Field, valErr.Message)
        }
    }
}

```

The custom `ValidationError` provides structured error information. `fmt.Errorf` wraps these errors while adding additional context.

## Error with position information

`fmt.Errorf` can include caller information in errors. This example shows how to create errors with file and line number details.

position_error.go

```go

package main

import (
    "fmt"
    "runtime"
)

func doSomething() error {
    _, file, line, _ := runtime.Caller(1)
    return fmt.Errorf("error at %s:%d - invalid operation", file, line)
}

func main() {
    err := doSomething()
    if err != nil {
        fmt.Println("Error:", err)
    }
}

```

The `runtime.Caller` function gets the caller's file and line number. This information is included in the formatted error message.

## Conditional error formatting

`fmt.Errorf` can be used with conditional logic to create different error messages. This example shows dynamic error message generation.

conditional_error.go

```go

package main

import (
    "fmt"
    "strings"
)

func processInput(input string) error {
    if len(input) == 0 {
        return fmt.Errorf("input error: empty string provided")
    }
    
    if strings.Contains(input, "error") {
        return fmt.Errorf("input error: contains forbidden word 'error'")
    }
    
    if len(input) > 100 {
        return fmt.Errorf("input error: length %d exceeds maximum of 100", len(input))
    }
    
    return nil
}

func main() {
    tests := []string{"", "test with error", strings.Repeat("a", 101)}
    
    for _, test := range tests {
        err := processInput(test)
        if err != nil {
            fmt.Printf("'%s' → %v\n", test, err)
        }
    }
}

```

Different error conditions generate different formatted messages. The function returns appropriate errors based on input validation.

## Source

[Go fmt package documentation](https://pkg.go.dev/fmt#Errorf)

This tutorial covered the `fmt.Errorf` function in Go with practical examples of formatted error creation and wrapping.