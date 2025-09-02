
https://golangdocs.com/integers-in-golang

Go supports integer data types extensively. Now, we will see what are those.

## Signed integers in Go

Signed integer types supported by Go is shown below.

- **int8 (8-bit signed** integer whose range is **-128 to 127)**
- **int16 (16-bit signed** integer whose range is **-32768 to 32767)**
- **int32 (32-bit signed** integer whose range is **-2147483648 to 2147483647)**
- **int64 (64-bit signed** integer whose range is **-9223372036854775808 to 9223372036854775807)**

## Unsigned integers in Go

- **uint8 (8-bit unsigned** integer whose range is **0 to 255 )**
- **uint16 (16-bit** **unsigned** integer whose range is **0 to 65535 )**
- **uint32 (32-bit** **unsigned** integer whose range is **0 to 4294967295 )**
- **uint64 (64-bit unsigned** integer whose range is **0 to 18446744073709551615 )**

## Integer Overflow in Golang

If you assign a type and then use a number larger than the types range to assign it, it will fail. Below is a program trying just that.

|                                                                                               |                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11 | `package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `var x uint8`<br><br>        `fmt.Println(``"Throws integer overflow"``)`<br><br>    `x =` `267`       `// range of uint8 is 0-255`<br><br>`}` |

## Type conversion in Golang

If you convert to a type that has range lower than your current range, data loss will occur. We do typecast by directly using the name of the variable as a function to convert types.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `var x int32`<br><br>    `var y uint32`     `// range 0 to 4294967295`<br><br>    `var z uint8`      `// range 0 to 255`<br><br>    `fmt.Println(``"Type Conversion"``)`<br><br>    `x =` `26700`<br><br>    `y = uint32(x)`       `// data preserved because number is inside range`<br><br>    `z = uint8(x)`        `// data loss due to out of range conversion`<br><br>    `fmt.Println(y, z)`   `// prints 26700 76`<br><br>`}`|

Post navigation

