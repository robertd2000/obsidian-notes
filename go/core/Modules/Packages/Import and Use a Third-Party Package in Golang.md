https://thenewstack.io/import-and-use-a-third-party-package-in-golang/

Like most compiled programming languages, the [Go programming language](https://thenewstack.io/golang-co-creator-rob-pike-what-go-got-right-and-wrong/) makes it possible to use [external libraries](https://thenewstack.io/how-golang-manages-its-security-supply-chain/) and other pre-packaged tools. 

For example, with the tried and true Hello World application, we call the _main_ package and import the [fmt package](https://pkg.go.dev/fmt) like this:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|package main<br><br>import "fmt"<br><br>func main() {<br><br>    fmt.Println("Hello, New Stack")<br><br>}|

  
Without the ability to call those packages, you’d have to write every single line of code, which would vastly complicate the process. That simple five-line application could all of a sudden turn into a hundred-line application.

Given the goal is to work smarter and not harder, you do _not_ want to have to deal with all of that extra programming — especially since it’s already been done. And, in the case of _main_ and _fmt_, they’re built into the Golang application. 

Handy.

The _main_ package is special because it is used to make a program executable. By using main, you’re telling the Go compiler that the program should be compiled as an executable and not a library.

The _fmt_ package is important because it allows the formatting of basic strings and values such that they can be printed to the standard output, collect user input from the console, or write input to a file.

TRENDING STORIES

1. [The Cautionary Tale of the Browser Wars and Why Business Transformations Often Fail](https://thenewstack.io/the-cautionary-tale-of-the-browser-wars-and-why-business-transformations-often-fail/)
2. [Python for Beginners: When and How to Use Tuples](https://thenewstack.io/python-for-beginners-when-and-how-to-use-tuples/)
3. [Best Terminal Applications for Development](https://thenewstack.io/best-terminal-applications-for-development/)
4. [Code Anywhere: Turn Your Android Tablet Into a Dev Machine](https://thenewstack.io/code-anywhere-turn-your-android-tablet-into-a-dev-machine/)
5. [RailsConf and DHH Go Their Separate Ways](https://thenewstack.io/railsconf-and-dhh-go-their-separate-ways/)

There are a wide array of possible [packages that can be used for Golang](https://thenewstack.io/golang-how-to-use-library-packages/). You can go to the [Golang package search tool](https://pkg.go.dev/) and search through the repository of packages. Let’s say, for example, you want to write a Go application that will print out a random, pithy quote. You might think that would be challenging but since there’s already a package for this, you don’t have to worry about coding everything for the app.

What you do have to take care of before you can successfully compile your application is adding the third-party package. Fortunately, that’s not much of a challenge — you just have to know how it’s done.

Let me show you.

First, let me show you the application we’re going to run. Create a new .go file with the command:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">nano quote.go</span>|

  
In that file, paste the following:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11|package main<br><br>import "fmt"<br><br>import "rsc.io/quote"<br><br>func main() {<br><br> fmt.Println("Your pithy quote of the day is:\n")<br><br> fmt.Println(quote.Go())<br><br>}|

  
Once compiled, the above application will print the line:

_Don’t communicate by sharing memory, share memory by communicating._

If you were to attempt to compile the above without first adding the quote package, you’d receive an error.

How do you get around that? You first have to initialize the quote package, which is done with the command:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go mod init quote</span>|

  
Once you’ve done that, you can then download the package with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go mod tidy</span>|

  
Now, compile your application with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go build quote.go</span>|

  
This will create an executable named _call_. Run the app with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">./quote</span>|

  
You’ll see the output I listed above.

Let’s go a bit deeper. 

What we’re going to do now is create our own module and then call it from an application.

First, create a new directory structure with the command:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">mkdir -p projects/mymodule</span>|

  
Change into the modules directory with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">cd projects/mymodule</span>|

  
Initialize modules with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go mod init mymodule</span>|

  
You’ll find a new _go.mod_ file has been added.

Next, create a _main.go_ file with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">nano main.go</span>|

  
In that file, paste the following code:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|package main<br><br>import "fmt"<br><br>func main() {<br><br>fmt.Println("Hello, New Stack!")<br><br>}|

  
Pretty straightforward.

Save and close the file. 

You can now test-run the initial code with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go run main.go</span>|

  
You should see the following output:

_Hello, New Stack!_

Great.

Now, we’re going to create a new package that we’ll then call from our _main.go_ file. Still, in the mymodule directory, create a new directory with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">mkdir mypackage</span>|

  
Create a new .go file with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">nano mypackage/mypackage.go</span>|

  
In that file, paste the following:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|package newpackage<br><br>import "fmt"<br><br>func NSHello() {<br><br>             fmt.Println("Hello, New Stack! You've just called your first module!")<br><br>}|

  
As you can see, we’ve created a function, called NSHello() which prints the line “Hello, New Stack!” You’ve just called your first module!

Save and close the file. 

Next, we need to call our new package from within the main.go file. Open that file for editing with the command:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">nano /main.go</span>|

  
Currently, the file looks like this:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|package main<br><br>import "fmt"<br><br>func main() {<br><br>        fmt.Println("Hello, New Stack!")<br><br>}|

  
What we have to do is import the new package we just created. First, change the import section to:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|import (<br><br>       "fmt"<br><br>       "mymodule/mypackage"<br><br>)|

  
Next, change the func main() section to look like this:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5|func main() {<br><br>        fmt.Println("Hello, New Stack!")<br><br>        mypackage.NSHello()<br><br>}|

  
What we’ve done in this file is import our new package (with the line _mymodule/mypackage_) and then use it in conjunction with our _NSHello()_ function we defined in _mypackage.go_. 

The entire file looks like this:  

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12|package main<br><br>import (<br><br>       "fmt"<br><br>       "mymodule/mypackage"<br><br>)<br><br>func main() {<br><br>        fmt.Println("Hello, New Stack!")<br><br>        mypackage.PrintHello()<br><br>}|

  
Save and close the file.

Run the entire application with:  

|   |   |
|---|---|
|1|<span style="font-weight: 400;">go run main.go</span>|

  
The output should be:

_Hello, New Stack!  
__Hello, New Stack! You’ve just called your first module!_

Or, you can build an executable with the command: