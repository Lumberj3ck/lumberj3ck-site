+++
title = 'Go Packages'
date = 2024-05-09T10:49:00+02:00
draft = false
summary = "Discover how Go's package system elegantly simplifies your code structure. Learn how files sharing the same package name within a directory seamlessly access each other's variables, functions, and types, promoting streamlined code and reducing the need for complex import statements."
+++

In Go, packages provide a powerful way to organize and structure your code. A key feature is that files within the same package have direct access to each other's variables, functions, and types. This promotes seamless collaboration between different parts of your codebase. Let's explore how this works in practice.

If two files share the same **package name** within the same directory, they can access each other's names, such as variables and functions. We can execute **package** using the following command:

{{< highlight bash >}}
  go run *.go
{{< / highlight >}}

All files within a directory sharing the same package name effectively {{< emphasize >}} share a namespace.{{< / emphasize >}}

{{< highlight bash >}}
  ├── calculate
  │   ├── solution_runner.go
  │   └── types.go
{{< / highlight >}}

In this example, importing the calculate package allows us to use all the names defined in both **solution_runner.go** and **types.go**.

## How to import a package 

Let's assume we have the following folders structure. Here calculate  and eiler folders are go packages.  

{{< highlight bash >}}
  ├── calculate
  │   ├── solution_runner.go
  │   └── types.go
  ├── eiler
  │   ├── eiler_10
  │   │   └── eiler_10.go
  │   ├── eiler_6
  │   │   └── eiler_6.go
  │   ├── eiler_7
  │   │   └── eiler_7.go
  │   ├── eiler_8
  │   │   └── eiler_8.go
  │   └── eiler_9
  │       └── eiler_9.go
  └── go.mod
{{< / highlight >}}

To import names from the eiler package into solution_runner, we need to create a go.mod file for our project.

{{< highlight bash >}}
  go mod init my-package-name
{{< / highlight >}}

After running this command inside of folder we can see **go.mod** file

{{< highlight go >}}
module my-package-name

go 1.22.2
{{< / highlight >}}

All import paths should be relative to this path specified in go.mod.

**File: calculate/solution_runner.go**

{{< highlight go >}}
package main

import "fmt"
import so "my-package-name/eiler/eiler_6"

func main() {
  fmt.Println(so.Solution(2000_000))
}
{{< / highlight >}}

The location of the main file importing these modules doesn't matter; only the path to the imported directory is important. For instance, if we have a package named **eiler_10**, all files within that directory can share variables and object names.

**File: eiler/eiler_6/eiler_6.go**

{{< highlight go >}}
package eiler_6

import "math"

func Solution(n Element) int {
  sum_of_quad := float64(0)
  sum_of_numbers := Element(0)
  for i := Element(1); i <= n; i++ {
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

The Element type, defined in **types_eiler_6.go**, is used within **eiler_6.go**. When we import the **eiler_6** package into our main function, we don't encounter an error about Element being undefined. This is because it's imported as part of the package.

We can also run everything as a package. If our main package has types defined in another file within that package, we can execute it like so:

{{< highlight bash >}}
  go run .
  go run calculate // if inside of parent directory
  go run main.go
{{< / highlight >}}

If we attempt to run only one file from the package, we'll get an error indicating that an imported name is undefined.

## How to Use Local Modules Inside of Other Modules
Let's create a module:

{{< highlight bash >}}
mkdir mymodule
cd mymodule
{{< / highlight >}}

Inside mymodule, let's create a go.mod file to mark this directory as a module for Go.

{{< highlight bash >}}
go mod init mymodule
{{< / highlight >}}

Within go.mod:

{{< highlight go >}}
module mymodule

go 1.22.2
{{< / highlight >}}

{{< emphasize >}}mymodule{{< / emphasize >}} is an alias we can use when importing this module.

Let's create inside of **mymodule** a package named calc to define some functionality.

{{< highlight bash >}}
mkdir calc
cd calc
touch math.go
{{< / highlight >}}

Inside math.go:

{{< highlight go >}}
package calc

func AddInt(a, b int) int {
  return a + b
}
{{< / highlight >}}

To test this module, let's create a main module alongside mymodule.

{{< highlight bash >}}
cd ../..
mkdir main
cd main
go mod init main
touch main.go
{{< / highlight >}}

After all we'll have the following folders structure:
{{< highlight bash >}}
├── main
│   ├── go.mod
│   ├── main.go
│   └── utils.go
└── mymodule
    ├── calc
    │   └── math.go
    └── go.mod
{{< / highlight >}}


Inside main.go:


{{< highlight go >}}
package main

import (
  "fmt"
  "mymodule/calc"
)

func main() {
  fmt.Println(calc.AddInt(1, 2))
}
{{< / highlight >}}


**Inside of main directory**
{{< highlight bash >}}
go run .
{{< / highlight >}}

If we try to run this, we'll get an error:

{{< highlight bash >}}
package mymodule/calc is not in GOROOT (/usr/local/go/src/mymodule/calc)
{{< / highlight >}}

{{< blockquote >}}
This error occurs because we haven't specified where Go should find the mymodule module within main/go.mod. Go attempted to search in GOROOT but couldn't find it and, since the module name doesn't resemble a URL, emitted this error.
{{< / blockquote >}}

We need to explicitly tell the Go compiler where mymodule is located. If the module is only on our local machine, we use the replace directive.

{{< highlight go >}}
module main

go 1.22.2

// location of mymodule relative to the main module
replace mymodule => ../mymodule
{{< / highlight >}}

Let's test again:

Inside the main directory

{{< highlight bash >}}
go run  .
{{< / highlight >}}


We'll get almost equivalent error, but now we just need to get the module:

{{< highlight bash >}}
package mymodule/calc is not in GOROOT (/usr/local/go/src/mymodule/cal)
module mymodule provides package mymodule/calc and is replaced but not required; to add it:
go get mymodule
{{< / highlight >}}

Let's run the suggested command:

{{< highlight bash >}}
go get mymodule
{{< / highlight >}}

Inside main/go.mod, we now have a new line:

{{< highlight bash >}}
require mymodule v0.0.0-00010101000000-000000000000 // indirect
{{< / highlight >}}

This indicates the specific version of the mymodule module that the Go compiler will use for building.

Running main.go one more time should now work successfully!
