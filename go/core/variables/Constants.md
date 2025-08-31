https://webreference.com/go/basics/constants/

# Working with Constants in Go Programming

Constants in Go are immutable values that are known at compile time. This guide covers everything you need to know about working with constants in Go.

## Declaring Constants

### Basic Constants

```go
// Single constant
const Pi = 3.14159

// Multiple constants
const (
    StatusOK    = 200
    StatusError = 500
)

// Typed constants
const (
    Timeout time.Duration = 30 * time.Second
    MaxSize int          = 1024
    Debug   bool         = false
)
```

### Untyped Constants

```go
const (
    // These constants are untyped
    Pi    = 3.14159
    Hello = "Hello, World"
    Yes   = true
)

// Constants can be used with different compatible types
var f float64 = Pi
var f32 float32 = Pi
var c complex128 = Pi
```

## Using iota

### Basic iota

```go
const (
    Sunday = iota    // 0
    Monday          // 1
    Tuesday         // 2
    Wednesday       // 3
    Thursday        // 4
    Friday         // 5
    Saturday        // 6
)
```

### Advanced iota Patterns

```go
const (
    // Bit shifting
    KB = 1 << (10 * iota)  // 1 << (10 * 0) = 1
    MB                     // 1 << (10 * 1) = 1024
    GB                     // 1 << (10 * 2) = 1048576
    TB                     // 1 << (10 * 3) = 1073741824
)

const (
    // Skip values
    _           = iota             // 0
    Debug Level = 1 << iota       // 1 << 1
    Info                          // 1 << 2
    Warning                       // 1 << 3
    Error                        // 1 << 4
)

const (
    // Offset and multiplier
    Offset = 2 * iota + 1  // 1
    _                      // 3
    Value                  // 5
    _                      // 7
    Result                 // 9
)
```

## Constant Types

### Numeric Constants

```go
const (
    // Integer constants
    MaxInt     = 9223372036854775807
    MinInt     = -9223372036854775808
    
    // Floating-point constants
    Pi         = 3.14159265359
    E          = 2.71828182845
    
    // Complex constants
    I          = 1i
    TwoI       = 2i
)
```

### String Constants

```go
const (
    // String constants
    Prefix     = "go_"
    Suffix     = "_temp"
    EmptyStr   = ""
    
    // Raw string literals
    MultiLine  = `Line 1
                  Line 2
                  Line 3`
)
```

### Boolean Constants

```go
const (
    // Boolean constants
    True       = true
    False      = false
    Debug      = true
)
```

## Constant Expressions

```go
const (
    // Constant expressions
    Hundred    = 10 * 10
    Million    = Hundred * Hundred * Hundred
    
    // String concatenation
    FullName   = "John" + " " + "Doe"
    
    // Boolean expressions
    IsValid    = true && !false
)
```

## Best Practices

### 1. Naming Conventions

```go
// Good: Clear, descriptive names
const (
    MaxConnections   = 100
    DefaultTimeout   = 30 * time.Second
    DatabaseURL     = "postgres://localhost:5432"
)

// Avoid: Unclear names
const (
    Max   = 100     // Too vague
    TO    = 30      // Too short
    URL   = "..."   // Too generic
)
```

### 2. Grouping Constants

```go
// Group related constants
const (
    // HTTP Status codes
    StatusOK           = 200
    StatusCreated      = 201
    StatusAccepted     = 202
    
    // Error codes
    StatusBadRequest   = 400
    StatusUnauthorized = 401
    StatusForbidden    = 403
)

// Configuration constants
const (
    MaxRetries        = 3
    RetryDelay        = 5 * time.Second
    RequestTimeout    = 30 * time.Second
)
```

### 3. Type Safety

```go
// Use typed constants for type safety
const (
    Timeout time.Duration = 30 * time.Second
    MaxSize int          = 1024 * 1024
)

// Use untyped constants for flexibility
const (
    Pi    = 3.14159
    Debug = true
)
```

## Common Patterns

### 1. Enumerated Constants

```go
type Direction int

const (
    North Direction = iota
    East
    South
    West
)

func (d Direction) String() string {
    return [...]string{"North", "East", "South", "West"}[d]
}
```

### 2. Bitmask Constants

```go
const (
    Read Permission = 1 << iota  // 1
    Write                        // 2
    Execute                      // 4
)

// Usage
func hasPermission(p Permission) bool {
    return p&Write != 0
}
```

### 3. Version Constants

```go
const (
    Major = 1
    Minor = 2
    Patch = 3
    
    Version = Major*10000 + Minor*100 + Patch
    VersionString = fmt.Sprintf("%d.%d.%d", Major, Minor, Patch)
)
```

## Performance Considerations

1. Constants are evaluated at compile time
2. No runtime overhead for constant expressions
3. Constants can help compiler optimizations

## Common Mistakes

### 1. Mutable Values

```go
// Wrong: Trying to modify constants
const MaxSize = 1024
MaxSize = 2048  // Compilation error

// Right: Use variables for mutable values
var maxSize = 1024
maxSize = 2048  // OK
```

### 2. Non-constant Expressions

```go
// Wrong: Non-constant expressions
const Timeout = time.Now()  // Error: not a constant

// Right: Constant expressions
const Timeout = 30 * time.Second
```