+++
title = 'Go Packages'
date = 2024-05-09T10:49:00+02:00
draft = false
summary = "Discover how Go's package system elegantly simplifies your code structure.  Learn how files sharing the same package name within a directory can effortlessly access each other's variables, functions, and types. This promotes streamlined code and reduces the need for complex import statements."  
+++

In Go, packages provide a powerful way to organize and structure your code. One key feature is that files within the same package can directly access each other's variables, functions, and types. This promotes seamless collaboration between different parts of your codebase. Let's see how this works in practice.  

If two files have same **package name** and they are in same directory they can access variables and names from each other names, e.g share names
We can run this thing like so
{{< highlight bash  >}}
  go run *.go 
{{< / highlight  >}}

All the package name under the directory name is sharing all the name in all those files. 

{{< highlight bash  >}}
  ├── calculate
  │   ├── solution_runner.go
  │   └── types.go
{{< / highlight  >}}

so if we import calculate we can use all names from both files solution_runner.go and types.go

{{< highlight bash  >}}
  ├── calculate
  │   ├── solution_runner.go
  │   └── types.go
  ├── eiler
  │   ├── eiler_10
  │   │   └── eiler_10.go
  │   ├── eiler_6
  │   │   └── eiler_6.go
  │   ├── eiler_7
  │   │   └── eiler_7.go
  │   ├── eiler_8
  │   │   └── eiler_8.go
  │   └── eiler_9
  │       └── eiler_9.go
  └── go.mod
{{< / highlight  >}}

If we want to import inside of solution runner something from eiler we need to create for all of this things a go.mod file 

{{< highlight go >}}
module learning_go/katas

go 1.22.2
{{< / highlight >}}

Every path for imports should be relative to this path in go mod 

**File: calculate/solution_runner.go**
{{< highlight go >}}
package main

import "fmt"
import so "learning_go/katas/eiler/eiler_6"


func main(){
    fmt.Println(so.Solution(2000_000))
    // fmt.Println(randomNumber())
}
{{< / highlight >}}

Doesn't matter where the main file which imports all those modules is located, matters only path for directory which we import
if for example we have a package eiler_10 all the files inside of this directory can have shared variables and object names  

**File: eiler/eiler_6/eiler_6.go**
{{< highlight go >}}
package eiler_6

import "math"
// import "fmt"

func Solution(n Element) int{
    sum_of_quad := float64(0)
    sum_of_numbers := Element(0)
    for i := Element(1); i <= n; i++{
        sum_of_quad += math.Pow(float64(i), 2)  
        sum_of_numbers += i
    }
    quad_of_sum := math.Pow(float64(sum_of_numbers), 2)
    return int(quad_of_sum - sum_of_quad)
}
{{< / highlight >}}

**File: eiler/eiler_6/types_eiler_6.go**
{{< highlight go >}}
package eiler_6


type Element int
	
{{< / highlight >}}

**Element** type is used inside of  **eiler_6**.**go** and we import **eiler_6** package inside of main function we don't see the error that **Element** type is not defined.  Because it imported as a package.

And we also can run everything as a package, and if for example some main package has types which 
are defined in another file inside of main package we can do so:

{{< highlight bash >}}
  go run .
  go run calculate // if inside of parent directory
  go run main.go
{{< / highlight >}}


If we instead will run only one file out of this package we will get an error that imported name is undefined.

## How to use local modules inside of other modules
Let's create a module:
{{< highlight bash >}}
  mkdir mymodule 
  cd mymodule
{{< / highlight >}}

Inside of math_module let's create go mod file, this will mark for go that this directory is a module. 

{{< highlight bash >}}
  go mod init mymodule
{{< / highlight >}}

Inside of go.mod

{{< highlight go >}}
module mymodule

go 1.22.2
{{< / highlight >}}

mymodule inside of "module mymodule" is alias which we can use when import this module. 
Lets create package calc where we would declare some functionality. 

{{< highlight bash >}}
  mkdir calc
  cd calc
  touch math.go
{{< / highlight >}}

Inside of math.go

{{< highlight go >}}
package calc


func AddInt(a, b int) int{
    return a + b
}
{{< / highlight >}}

For testing this module, let's create `main` module near `mymodule` 

{{< highlight bash >}}
  cd ../..
  mkdir main
  cd main
  go mod init main
  touch main.go
{{< / highlight >}}

Inside of main.go
{{< highlight go >}}
package main

import (
    "fmt"
    "mymodule/calc"
)


func main(){
    fmt.Println(calc.AddInt(1, 2))
}
{{< / highlight >}}

Try to run and get an error 
{{< highlight bash >}}
  # Inside of main directory
  go run .
{{< / highlight >}}

{{< highlight bash >}}
  package mymodule/calc  is not in std (/usr/local/go/src/mymodule/calc)
{{< / highlight >}}

{{< blockquote >}}
 The problem here, that we didn't declare anything inside of `main/go.mod` about where go should search for this module `mymodule`.  Go tried to search inside of GOROOT, when didn't found saw that name of our module doesn't look like url and emited following error. 
{{< / blockquote >}}

We need to explicitly say to go compiler where is `mymodule` located. If module is only on our local machine, we need to use `replace` directive. 

{{< highlight go >}}
module main

go 1.22.2

// location of mymodule relatively to main module
replace mymodule => ../mymodule
{{<  / highlight >}}

Let's test one more time

**Inside of main directory**

{{<   highlight bash >}}
  go run .
{{<  / highlight >}}

{{< highlight bash >}}
  package mymodule/calc  is not in std (/usr/local/go/src/mymodule/cal)
  module mymodule provides package mymodule/calc and is replaced but not required; to add it:
  go get mymodule
{{< / highlight >}}

Let's run the command:

{{<   highlight bash >}}
  go get mymodule
{{<  / highlight >}}

Inside of main/go.mod we got new string: 
{{<   highlight bash >}}
  require mymodule v0.0.0-00010101000000-000000000000 // indirect 
{{<  / highlight >}}

It points to which exactly version of this module go compiler will use for build.

Run the main.go one more time. Hooray, now it's working!

