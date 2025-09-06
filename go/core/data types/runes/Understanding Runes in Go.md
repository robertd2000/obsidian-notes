https://dev.to/jeseekuya/understanding-runes-in-go-4ie5

**Introduction to Runes**  
The American Standard Code for Information Interchange(ASCII) is a character set representing uppercase letters, lowercase letters, digits, and various punctuations and device-control characters. The issue with the ASCII is it does not include most characters used in the world like Chinese characters and other character sets used by different languages. This limitation led to the invention of Unicode characters. Unicode is a superset of ASCII; It contains all the characters present in today's global writing system and it assigns each character a special Unicode Code Point. This Unicode Code Point is known as runes in Golang.

Below is a representation of the ASCII table.  

```
Oct   Dec   Hex   Char                        Oct   Dec   Hex   Char
────────────────────────────────────────────────────────────────────────
000   0     00    NUL '\0' (null character)   100   64    40    @
001   1     01    SOH (start of heading)      101   65    41    A
002   2     02    STX (start of text)         102   66    42    B
003   3     03    ETX (end of text)           103   67    43    C
004   4     04    EOT (end of transmission)   104   68    44    D
005   5     05    ENQ (enquiry)               105   69    45    E
006   6     06    ACK (acknowledge)           106   70    46    F
007   7     07    BEL '\a' (bell)             107   71    47    G
010   8     08    BS  '\b' (backspace)        110   72    48    H
011   9     09    HT  '\t' (horizontal tab)   111   73    49    I
012   10    0A    LF  '\n' (new line)         112   74    4A    J
013   11    0B    VT  '\v' (vertical tab)     113   75    4B    K
014   12    0C    FF  '\f' (form feed)        114   76    4C    L
015   13    0D    CR  '\r' (carriage ret)     115   77    4D    M
016   14    0E    SO  (shift out)             116   78    4E    N
017   15    0F    SI  (shift in)              117   79    4F    O
020   16    10    DLE (data link escape)      120   80    50    P
021   17    11    DC1 (device control 1)      121   81    51    Q
```

A Unicode Code Point uniquely identifies a character in Unicode, where runes represent these characters as numbers. For instance, rune 32 denotes a space character. In Golang, `UTF-8` encoding is utilized for characters (Unicode code points), represented by a sequence of 1 to 4 bytes. This makes runes the preferred method for string manipulation in Golang, facilitating direct interaction with characters. As software evolves, developers increasingly use runes to manage Unicode complexities, ensuring applications support multilingualism, emojis, and other Unicode functionalities.

In Golang, a byte, an alias for uint8, spans 1 to 255, while a rune, aliasing `int32`, signifies a Unicode code point, with characters encoded using `UTF-8`. Runes empower developers to engage directly with characters.  
**Working with Runes in Golang**  
There are different ways to initialize a rune in Golang. Let's use the Japanese Yen as an example. The Unicode code point of the Yen is `U+00A5.`  
To initialize the yen, we can do so directly with the character or the Unicode representation of the character. This representation can be as an integer or a hexadecimal. The yen can also be initialized by an escape sequence. This escapes the U and + of the Unicode code point making it to be `\u00A5.` The same escape as an octal escape will be `\245.`  

```go
package main


func main() {
   yen := '¥'                  //initializing with the rune literal
   japYen := 165               //initializing with an integer
   jYen := 0x00A5              //initializing with a hex
   japanYen := '\u00A5'        //initializing with escape sequence
   japnYen := '\245'
}

```

To print the rune as a character. You can use `fmt.Printf` with the `%c` or type cast the rune variable to a string.  

```go
package main


import "fmt"


func main() {
   yen := '¥'           // initializing with the rune literal
   japYen := 165        // initializing with an integer
   jYen := 0x00A5       // initializing with a hex
   japanYen := '\u00A5' // initializing with escape sequence
   japnYen := '\245'


   fmt.Printf("%c\n", yen)
   fmt.Printf("%c\n", japYen)
   // jYen is an int type, type cast it to rune then string to print the character.
   fmt.Println(string(rune(jYen)))
   fmt.Println(string(japanYen))
   fmt.Printf("%c\n", japnYen)
}
```

This program will print the `'¥'` character on the terminal as shown below.  

```
\test$ go run .
¥
¥
¥
¥
¥
\test$
```

Slicing a string will give you a byte. In string manipulation, that is not ideal because as stated earlier; strings in go are encoded as a sequence 1 t0 4 bytes.  
To get the individual characters in a string. Range over the string to get the runes.  

```go
package main


import "fmt"


func main() {
   str := "Happy Coding"


   // Range str to get the individual characters
   for _, v := range str {
       //This will print the characters one by one
       fmt.Println(string(v))
   }
}

```

The above code will print the individual characters as follows;  

```
/test$ go run .
H
a
p
p
Y


C
o
d
i
n
g
/test$
```

To get the first character in a string. Range over, slicing it will give you the first byte, not the first character.  

```go
package main


import "fmt"


func main() {
  str := "💞 code"


  // slicing a string gives you a byte not the character
  // not ideal for string manipulation
  fmt.Println(string(str[0]))


  // Range str to get the first character
  for i, v := range str {
     if i == 0 {
        fmt.Println(string(v))
     }
  }
}

```

The first printing will give you this character ð. This is not the first character that you wanted. Ranging the string gives you the right character as shown below.  

```
/test$ go run .
ð
💞
/test$

```

To better understand how to work with runes, let us develop a function that works as a Stconv.Atoi package where it takes a string and returns an integer type.  

```go
// Atoi takes a string and returns an int with an error message
func Atoi(str string) (int, error) {
   var num int
   neg := false //check for negative numbers
   // str[0] gives us a byte. In this case the first byte
   // since the negative sign is an Ascii character. It is encoded as 1 byte
   //non-ASCII characters are encoded as sequences of 2 to 4 bytes
   //Slicing, or indexing a string gives you a byte, not a rune
   if string(str[0]) == "-" {
       neg = true
       str = str[1:]
   }


   // To get the runes in the string, range over the string
   //Ranging a string gives you a rune, not a byte
   for _, v := range str {
       // In the ascii table. The characters 0 to 9 are arranged in order
       if v >= '0' && v <= '9' {
           // Let's say v is the character 5 with the rune of 54
           // subtracting 54 from the rune of 0 which is 49
           // gives you 5
           num = num*10 + int(v-'0')
       } else {
           // characters with runes less than 49 and
           // runes greater than 58 will return 0 and a
           // parsing error
           return 0, fmt.Errorf(" parsing %v: invalid syntax", str)
       }
   }
   if neg {
       num *= -1
   }
   return num, nil
}

```

Let’s write a program to test the function.  

```go
package main


import "fmt"


func main() {
  // I'm not checking for errors
  num, _ := Atoi("45")
  num1, _ := Atoi("89")
  // Print the sum of num and num1
  fmt.Println(num + num1)


  // I am checking for errors
  num2, err2 := Atoi("ax56")
  if err2 != nil {
     fmt.Println(err2)
  }
  // num2 will be zero
  fmt.Println(num2)
}

```

The above program will output the following;  

```
/test$ go run .
134
parsing ax56: invalid syntax
0
/test$
```

In summary, runes in Golang are fundamental for handling Unicode characters effectively within strings. They provide a way to represent and process individual characters in a way that is essential for robust and internationalized software development.