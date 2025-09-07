https://thelinuxcode.com/golang-rune/

Friend, have you ever needed to process text from multiple languages within your Go programs? Dealing with Unicode encodings and variable width characters can be challenging!

This is where **runes** come to the rescue.

In this comprehensive guide, we will demystify runes in Go and how they enable seamless Unicode and UTF-8 text handling.

You will gain clarity on:

- What exactly are runes?
- Creating, printing and converting runes
- Leveraging runes for text processing
- Best practices for utilizing runes

By the end, you will have confidence in wielding the power of runes within your own code!

## What Are Runes in Go?

To understand runes, we must first take a quick detour into how characters are encoded digitally.

**ASCII** was one of the initial character encoding standards, using 7 bits to represent 128 characters – focusing only on English letters, numbers, and symbols.

But ASCII could not handle other languages. Hence the **Unicode** standard was invented, assigning each character a unique number up to 1.1 million code points!

The common encoding of Unicode for the web is **UTF-8**, using 1-4 bytes for every character.

This brought complexity in processing textprogrammatically. To abstract this away, Go introduced **runes**.

_Runes are an alias for the int32 type, representing a Unicode code point._

We can now handle UTF-8 text through the rune datatype rather than directly with bytes!

## Creating and Printing Runes

Runes are created by simply enclosing any Unicode character with single quotes:

```go
r1 := ‘A‘
r2 := ‘©‘
r3 := ‘使‘ // Chinese character
```

We can print the numeric value using `%U`:

```go
fmt.Printf("Rune: %U\n", r1) // U+0041 
fmt.Printf("Rune: %U\n", r2) // U+00A9
```

In fact runes can come from any written language with Unicode representation:

```go
arabic := ‘ال‘      // Arabic
bangla := ‘স‘       // Bangla
tamil := ‘த‘       // Tamil
japanese := ‘あ‘ // Japanese
```

So by abstracting away encodings, runes provide a simple way to handle UTF-8 text in Go.

> Note: The quoted syntax prevents compile errors from special characters.

## Converting Strings to Runes

To process text as runes instead of raw bytes, we _cast_ strings to rune slices:

```go
str := "Señor" 

runeSlice := []rune(str) // Type []int32

fmt.Println(runeSlice) 
// [83 101 ñ 111 114]  
```

This conversion allocates memory based on length, taking ~2X bytes for UTF-8.

We can now iterate through the characters properly:

```go
for _, r := range []rune(str) {
  fmt.Printf("%c", r)  
}
// Prints Señor
```

And convert the runes back to a string:

```go
str2 := string([]rune{83, 101, 241, 111, 114})  
fmt.Println(str2) // Señor
```

So rune conversion unlocks text manipulation at the character level!

## Manipulating Text Using Runes

Runes open up many text processing methods like:

**Indexing** based on characters instead of bytes:

```
idx := strings.IndexRune("Señor", ‘ñ‘) // 3
```

**Trimming** properly for display:

```go
trimmed := strings.TrimRightFunc("¡Hola!", isNotLetter)
// ¡Hola
```

And **splitting** text at character boundaries:

```go
fmt.Println(strings.Split("こんにちは", "")) 
// [こ ん に ち は]
```

Furthermore, runes interoperate with byte streams:

```go
reader := transform.NewReader(bytes.NewReader([]byte("Señor")), charmap.Windows1252.NewDecoder())
runes, _, _ := transform.BytesToRunes(reader)  
```

So leveraging runes unlocks proper Unicode processing.

> Contrast this to error-prone byte array slicing and handling!

## Best Practices For Utilizing Runes

Here are some key guidelines as you leverage runes:

- Use runes when supporting languages beyond ASCII
- Be aware conversion to `[]rune` costs allocation
- Profile performance for high load services
- Understand Unicode encoding specifics

The following table summarizes the tradeoffs:

||Runes|Byte Arrays|
|---|---|---|
|Text Processing|Excellent|Prone to errors|
|Performance|Slower|Faster|
|Readability|Better|Messy indices|

In most cases ease of use will outweigh minor performance gains from byte arrays.

## Additional Functionality with Runes

Beyond conversions, the `unicode` package provides many helper methods for runes:

```go
import "unicode"

unicode.IsUpper(‘S‘) // true

unicode.ToLower(‘A‘) // ‘a‘

unicode.IsSpace(‘\u0085‘) // true
```

And custom data structures can embed runes:

```go
type RuneSet map[rune]struct{}

var vowels RuneSet = ...
```

So runes can be utilized in diverse ways.

## Conclusion

I hope this guide helped demystify runes for you! Here are some key takeways:

- Runes represent Unicode code points as `int32` aliases
- They unlock proper UTF-8 string processing
- Enable text manipulation at the character level
- Streamline dealing with complex variable width encodings

Leveraging runes will make working with real-world textual data much simpler. No more worrying about bytes and indices!

For more rune usage tips, check out the references below. Happy coding friend!

### References

- [https://blog.golang.org/strings](https://blog.golang.org/strings)
- [https://golangbot.com/rune/](https://golangbot.com/rune/)
- [https://LINGUAPRO.COM/go/golang-unicode-runes/](https://linguapro.com/go/golang-unicode-runes/)