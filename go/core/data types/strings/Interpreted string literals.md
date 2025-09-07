https://boldlygo.tech/archive/2023-01-30-lexical-elements-interpreted-string-literals/

Picking up where we left off last week, we’re discussing string literals. Last week we talked about [raw string literals](https://boldlygo.tech/archive/2023-01-27-lexical-elements-raw-string-literals/), which leaves us now with interpreted string literals. To recap:

> ## [String literals](https://go.dev/ref/spec#String_literals)
> 
> A string literal represents a [string constant](https://go.dev/ref/spec#Constants) obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.

So now on to the interpreted string literals, which you’re likely to use most of the time in your Go programming.

> Interpreted string literals are character sequences between double quotes, as in `"bar"`. Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in [rune literals](https://go.dev/ref/spec#Rune_literals) (except that `\'` is illegal and `\"` is legal), with the same restrictions. The three-digit octal (`\`_nnn_) and two-digit hexadecimal (\xnn) escapes represent individual _bytes_ of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual _characters_. Thus inside a string literal `\377` and `\xFF` represent a single byte of value `0xFF`=255, while `ÿ`, `\u00FF`, `\U000000FF` and `\xc3\xbf` represent the two bytes `0xc3` `0xbf` of the UTF-8 encoding of character U+00FF.

So let’s unpack this.

First, as you may recall from our discussion of rune literals ([here](https://boldlygo.tech/archive/2023-01-25-lexical-elements-rune-literals-pt-2/) and [here](https://boldlygo.tech/archive/2023-01-26-lexical-elements-rune-literals-pt-3/)), we have a number of options for escape sequences for special (interpreted) characters in our string literals.

Here we get to mix and match them as we see fit. This gives us a lot of flexibility in choosing how to represent a string in our code. Sometimes you may want a human readable version:

```golang
"日本語"
```

Or maybe you want to represent the same string using one of the explicit Unicode or UTF-8 representations:

```golang
"\u65e5\u672c\u8a9e"
"\U000065e5\U0000672c\U00008a9e"
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"
```

Further, you may recall that with `rune` literals, we cannot represent two code points as a single `rune`, even if they appear as a single character on string. We don’t have that limitation with string literals. This can be good or bad. As the spec points out:

> If the source code represents a character as two code points, such as a combining form involving an accent and a letter, the result will be an error if placed in a rune literal (it is not a single code point), and will appear as two code points if placed in a string literal.

So to summarize, if you want to draw attention to the fact that `ў` is two runes, you may want to use an interpreted string literal with the escaped Unicode codepoints:

```golang
"\u0443\u0306"
```

Now, let me get back to some of the bits I skipped in my explanation, for the sake of completeness:

> ```ebnf
> string_lit             = raw_string_lit | interpreted_string_lit .
> raw_string_lit         = "`" { unicode_char | newline } "`" .
> interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
> ```

> ```golang
> `abc`                // same as "abc"
> `\n
> \n`                  // same as "\\n\n\\n"
> "\n"
> "\""                 // same as `"`
> "Hello, world!\n"
> "日本語"
> "\u65e5本\U00008a9e"
> "\xff\u00FF"
> "\uD800"             // illegal: surrogate half
> "\U00110000"         // illegal: invalid Unicode code point
> ```

These examples all represent the same string:

> ```golang
> "日本語"                                 // UTF-8 input text
> `日本語`                                 // UTF-8 input text as a raw literal
> "\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
> "\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
> "\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
> ```

Quotes from _The Go Programming Language Specification, Version of June 29, 2022_