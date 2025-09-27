https://go.dev/doc/tutorial/create-module

This is the first part of a tutorial that introduces a few fundamental features of the Go language. If you're just getting started with Go, be sure to take a look at [Tutorial: Get started with Go](https://go.dev/doc/tutorial/getting-started.html), which introduces the `go` command, Go modules, and very simple Go code.

In this tutorial you'll create two modules. The first is a library which is intended to be imported by other libraries or applications. The second is a caller application which will use the first.

This tutorial's sequence includes seven brief topics that each illustrate a different part of the language.

1. Create a module -- Write a small module with functions you can call from another module.
2. [Call your code from another module](https://go.dev/doc/tutorial/call-module-code.html) -- Import and use your new module.
3. [Return and handle an error](https://go.dev/doc/tutorial/handle-errors.html) -- Add simple error handling.
4. [Return a random greeting](https://go.dev/doc/tutorial/random-greeting.html) -- Handle data in slices (Go's dynamically-sized arrays).
5. [Return greetings for multiple people](https://go.dev/doc/tutorial/greetings-multiple-people.html) -- Store key/value pairs in a map.
6. [Add a test](https://go.dev/doc/tutorial/add-a-test.html) -- Use Go's built-in unit testing features to test your code.
7. [Compile and install the application](https://go.dev/doc/tutorial/compile-install.html) -- Compile and install your code locally.

**Note:** For other tutorials, see [Tutorials](https://go.dev/doc/tutorial/index.html).

## Prerequisites[¶](https://go.dev/doc/tutorial/create-module#prerequisites)

- **Some programming experience.** The code here is pretty simple, but it helps to know something about functions, loops, and arrays.
- **A tool to edit your code.** Any text editor you have will work fine. Most text editors have good support for Go. The most popular are VSCode (free), GoLand (paid), and Vim (free).
- **A command terminal.** Go works well using any terminal on Linux and Mac, and on PowerShell or cmd in Windows.

## Start a module that others can use[¶](https://go.dev/doc/tutorial/create-module#start)

Start by creating a Go module. In a module, you collect one or more related packages for a discrete and useful set of functions. For example, you might create a module with packages that have functions for doing financial analysis so that others writing financial applications can use your work. For more about developing modules, see [Developing and publishing modules](https://go.dev/doc/modules/developing).

Go code is grouped into packages, and packages are grouped into modules. Your module specifies dependencies needed to run your code, including the Go version and the set of other modules it requires.

As you add or improve functionality in your module, you publish new versions of the module. Developers writing code that calls functions in your module can import the module's updated packages and test with the new version before putting it into production use.

1. Open a command prompt and `cd` to your home directory.
    
    On Linux or Mac:
    
    cd
    
    On Windows:
    
    cd %HOMEPATH%
    
2. Create a `greetings` directory for your Go module source code.
    
    For example, from your home directory use the following commands:
    
    mkdir greetings
    cd greetings
    
3. Start your module using the [`go mod init` command](https://go.dev/ref/mod#go-mod-init).
    
    Run the `go mod init` command, giving it your module path -- here, use `example.com/greetings`. If you publish a module, this _must_ be a path from which your module can be downloaded by Go tools. That would be your code's repository.
    
    For more on naming your module with a module path, see [Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module).
    
    $ go mod init example.com/greetings
    go: creating new go.mod: module example.com/greetings
    
    The `go mod init` command creates a go.mod file to track your code's dependencies. So far, the file includes only the name of your module and the Go version your code supports. But as you add dependencies, the go.mod file will list the versions your code depends on. This keeps builds reproducible and gives you direct control over which module versions to use.
    
4. In your text editor, create a file in which to write your code and call it greetings.go.
5. Paste the following code into your greetings.go file and save the file.
    
    package greetings
    
    import "fmt"
    
    // Hello returns a greeting for the named person.
    func Hello(name string) string {
        // Return a greeting that embeds the name in a message.
        message := fmt.Sprintf("Hi, %v. Welcome!", name)
        return message
    }
    
    This is the first code for your module. It returns a greeting to any caller that asks for one. You'll write code that calls this function in the next step.
    
    In this code, you:
    
    - Declare a `greetings` package to collect related functions.
    - Implement a `Hello` function to return the greeting.
        
        This function takes a `name` parameter whose type is `string`. The function also returns a `string`. In Go, a function whose name starts with a capital letter can be called by a function not in the same package. This is known in Go as an exported name. For more about exported names, see [Exported names](https://go.dev/tour/basics/3) in the Go tour.
        
        ![](https://go.dev/doc/tutorial/images/function-syntax.png)
    - Declare a `message` variable to hold your greeting.
        
        In Go, the `:=` operator is a shortcut for declaring and initializing a variable in one line (Go uses the value on the right to determine the variable's type). Taking the long way, you might have written this as:
        
        var message string
        message = fmt.Sprintf("Hi, %v. Welcome!", name)
        
    - Use the `fmt` package's [`Sprintf` function](https://pkg.go.dev/fmt/#Sprintf) to create a greeting message. The first argument is a format string, and `Sprintf` substitutes the `name` parameter's value for the `%v` format verb. Inserting the value of the `name` parameter completes the greeting text.
    - Return the formatted greeting text to the caller.

https://go.dev/doc/tutorial/call-module-code

# Call your code from another module

In the [previous section](https://go.dev/doc/tutorial/create-module.html), you created a `greetings` module. In this section, you'll write code to make calls to the `Hello` function in the module you just wrote. You'll write code you can execute as an application, and which calls code in the `greetings` module.

**Note:** This topic is part of a multi-part tutorial that begins with [Create a Go module](https://go.dev/doc/tutorial/create-module.html).

1. Create a `hello` directory for your Go module source code. This is where you'll write your caller.
    
    After you create this directory, you should have both a hello and a greetings directory at the same level in the hierarchy, like so:
    
    <home>/
     |-- greetings/
     |-- hello/
    
    For example, if your command prompt is in the greetings directory, you could use the following commands:
    
    cd ..
    mkdir hello
    cd hello
    
2. Enable dependency tracking for the code you're about to write.
    
    To enable dependency tracking for your code, run the [`go mod init` command](https://go.dev/ref/mod#go-mod-init), giving it the name of the module your code will be in.
    
    For the purposes of this tutorial, use `example.com/hello` for the module path.
    
    $ go mod init example.com/hello
    go: creating new go.mod: module example.com/hello
    
3. In your text editor, in the hello directory, create a file in which to write your code and call it hello.go.
4. Write code to call the `Hello` function, then print the function's return value.
    
    To do that, paste the following code into hello.go.
    
    package main
    
    import (
        "fmt"
    
        "example.com/greetings"
    )
    
    func main() {
        // Get a greeting message and print it.
        message := greetings.Hello("Gladys")
        fmt.Println(message)
    }
    
    In this code, you:
    
    - Declare a `main` package. In Go, code executed as an application must be in a `main` package.
    - Import two packages: `example.com/greetings` and the [`fmt` package](https://pkg.go.dev/fmt/). This gives your code access to functions in those packages. Importing `example.com/greetings` (the package contained in the module you created earlier) gives you access to the `Hello` function. You also import `fmt`, with functions for handling input and output text (such as printing text to the console).
    - Get a greeting by calling the `greetings` package’s `Hello` function.
5. Edit the `example.com/hello` module to use your local `example.com/greetings` module.
    
    For production use, you’d publish the `example.com/greetings` module from its repository (with a module path that reflected its published location), where Go tools could find it to download it. For now, because you haven't published the module yet, you need to adapt the `example.com/hello` module so it can find the `example.com/greetings` code on your local file system.
    
    To do that, use the [`go mod edit` command](https://go.dev/ref/mod#go-mod-edit) to edit the `example.com/hello` module to redirect Go tools from its module path (where the module isn't) to the local directory (where it is).
    
    1. From the command prompt in the hello directory, run the following command:
        
        $ go mod edit -replace example.com/greetings=../greetings
        
        The command specifies that `example.com/greetings` should be replaced with `../greetings` for the purpose of locating the dependency. After you run the command, the go.mod file in the hello directory should include a [`replace` directive](https://go.dev/doc/modules/gomod-ref#replace):
        
        module example.com/hello
        
        go 1.16
        
        replace example.com/greetings => ../greetings
        
    2. From the command prompt in the hello directory, run the [`go mod tidy` command](https://go.dev/ref/mod#go-mod-tidy) to synchronize the `example.com/hello` module's dependencies, adding those required by the code, but not yet tracked in the module.
        
        $ go mod tidy
        go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
        
        After the command completes, the `example.com/hello` module's go.mod file should look like this:
        
        module example.com/hello
        
        go 1.16
        
        replace example.com/greetings => ../greetings
        
        require example.com/greetings v0.0.0-00010101000000-000000000000
        
        The command found the local code in the greetings directory, then added a [`require` directive](https://go.dev/doc/modules/gomod-ref#require) to specify that `example.com/hello` requires `example.com/greetings`. You created this dependency when you imported the `greetings` package in hello.go.
        
        The number following the module path is a _pseudo-version number_ -- a generated number used in place of a semantic version number (which the module doesn't have yet).
        
        To reference a _published_ module, a go.mod file would typically omit the `replace` directive and use a `require` directive with a tagged version number at the end.
        
        require example.com/greetings v1.1.0
        
        For more on version numbers, see [Module version numbering](https://go.dev/doc/modules/version-numbers).
        
6. At the command prompt in the `hello` directory, run your code to confirm that it works.
    
    $ go run .
    Hi, Gladys. Welcome!
    

Congrats! You've written two functioning modules.