https://go.dev/ref/spec#Short_variable_declarations
A _short variable declaration_ uses the syntax:

ShortVarDecl = [IdentifierList](https://go.dev/ref/spec#IdentifierList) ":=" [ExpressionList](https://go.dev/ref/spec#ExpressionList) .

It is shorthand for a regular [variable declaration](https://go.dev/ref/spec#Variable_declarations) with initializer expressions but no types:

```

"var" IdentifierList "=" ExpressionList .

```


```go

i, j := 0, 10
f := func() int { return 7 }
ch := make(chan int)
r, w, _ := os.Pipe()  // os.Pipe() returns a connected pair of Files and an error, if any
_, y, _ := coord(p)   // coord() returns three values; only interested in y coordinate

```


Unlike regular variable declarations, a short variable declaration may _redeclare_ variables provided they were originally declared earlier in the same block (or the parameter lists if the block is the function body) with the same type, and at least one of the non-[blank](https://go.dev/ref/spec#Blank_identifier) variables is new. As a consequence, redeclaration can only appear in a multi-variable short declaration. Redeclaration does not introduce a new variable; it just assigns a new value to the original. The non-blank variable names on the left side of `:=` must be [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers).

```go

field1, offset := nextField(str, 0)
field2, offset := nextField(str, offset)  // redeclares offset
x, y, x := 1, 2, 3                        // illegal: x repeated on left side of :=

```


Short variable declarations may appear only inside functions. In some contexts such as the initializers for ["if"](https://go.dev/ref/spec#If_statements), ["for"](https://go.dev/ref/spec#For_statements), or ["switch"](https://go.dev/ref/spec#Switch_statements) statements, they can be used to declare local temporary variables.