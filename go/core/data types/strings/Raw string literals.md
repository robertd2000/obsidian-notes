https://boldlygo.tech/archive/2023-01-27-lexical-elements-raw-string-literals/

Now that I’ve bored you to death with more about `rune` literals than you ever asked, let’s take things up a level, and talk about strings.

> ## [String literals](https://go.dev/ref/spec#String_literals)
> 
> A string literal represents a [string constant](https://go.dev/ref/spec#Constants) obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.
> 
> Raw string literals are character sequences between back quotes, as in `` `foo` ``. Within the quotes, any character may appear except back quote. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes have no special meaning and the string may contain newlines. Carriage return characters (’\r’) inside raw string literals are discarded from the raw string value.

Although mentioned first in the spec, the raw string literal is less common than the interpreted string literal. It’s also a bit simpler. Raw strings are most useful in two scenarios:

1. Including newlines
    
    If you want your string to contain newlines, then a raw string literal is your friend. You may want this when building SQL strings for example:
    
    ```golang
    query := `SELECT *
              FROM users
              WHERE name = ?`
    ```
    
    This will produce the string, exactly as written, including newlines and leading spaces.
    
2. To avoid extraneous escaping
    
    The second common place to use the raw string literal is when your string would otherwise contain an onerous amount of escaping. For example, a regular expression to match a number with a decimal point might look like this, with an interpreted string literal:
    
    ```golang
    re := regexp.MustCompile("\\d+\\.\\d+")
    ```
    
    But with a raw string literal, it’s much more readable:
    
    ```golang
    re := regexp.MustCompile(`\d+\.\d+`)
    ```
    

As a special rule, to facilitate cross-platform compatibility ([I’m looking at you, Windows](https://en.wikipedia.org/wiki/Newline#History)), raw string literals will be stripped of any carraige return characters (i.e `\r`) in the text. Without this rule, two source file that appear identical (and indeed, may _be_ identical in [git](https://book.git-scm.com/docs/gitattributes/2.13.7#_text)) may produce different strings when executed on Windows versus Unix machines.

If you want the literal carriage returns to remain in your string, you have a few options:

1. Use an interpreted string, with literal carriage return escape sequences.
    
2. Encode the entire string using some other format, such as Base64.
    
3. Read the string from a file on disk (or embed it from disk)
    
4. Do some ugly mix-and-match between raw and interpreted string literals:
    
    ```golang
    mixedString := `This is a raw string literal` + "\r" +`
                    that includes literal carraige` + "\r" + `
                    returns at the end of each line.`+"\r" + `
                    `
    ```
    
    Probably don’t do that one.