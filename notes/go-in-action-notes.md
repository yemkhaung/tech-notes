# Notes for Go-in-action

Notes from book reading **Go In Action** by *Brian Kettlesin* and *Erik Saint Martin*

## Chapter 1 : Introducing Go

#### Go the *SUPER*

* Simple ways to embrace **concurrency**  that suits to modern computing environment (clusters/cloud computing, multi-core processors)
* Lightning-fast **compiler**
  - efficient dependency management
  - every file compiled ONLY ONCE
  - only imported sub package is compiled)
* **Packing system** for easy code sharing (for many teams developing many small packages, libraries)
* *Simple and Effective* **type-system** that rethinks the overhead of complex traditional *Object-oriented* development and enables code reuse
* * 
* Balances **rapid development** of high-leve languages(Ruby, Python) with **performant** low-level languages (C,C++)
* automatic memory management with built-in **Garbage Collector**

#### Development Speed

* compiler is smart and uses simple dependency resolution algorithms
* only build dependencies that are directly included in the program instead of transversing the dependencies of all the libraries as in Java, C, C++
* loose typing (dynamic types + type safety of static typing)
  * Do not need comprehensive test suites to check type bugs at runtime

#### Concurrency

* Effective tools to utilize multi-core resources
* *Goroutines* for concurrent processing (less memory and less code than threads)
  * Go runtime automatically schedule coroutines execution on logical processors (OS Thread)
  * A single OS thread can handle as a bucket for many Goroutines
  * Minimum overhead allows tens of thousands of goroutines to be spawned concurrently (which is quite common in Go)
* *Channels* as data structures for synchronized typed messaging between Go routines
  * Copy of data is **sequentially** passed around Goroutines that only one Goroutine can hold/modify the data at a time
  * Data is synchronized when it is exchanged via *Channels
  * Both parties of Goroutines can have gurantee of the data exchange
  * solves the problems of *locks and synchronization* in traditional multi-threaded programming
  * Does *not* provide *data access protection*. If *pointers* are passed via channels, each goroutines need to be synchronized to avoid updates from other parties

#### Go's Type System

* Flexible hierarchy-free type system
* Reuse code by **composition**. Types are **composed** of smaller types
* Model Behaviors not Types.
* For example, a `Truck` needs bahaviors of *driving*, *carry passengers*, *carry cargo*. 
  * In tranditional OOP, those behaviors might be tied to parent types `Vehicle > PassengerVehicle > Truck`. 
  * In Go types, There is only one type, `Truck` which is *composed* of behaviors `Drive`, `CarryPassengers`, `CarryCargos`. You can even reuse those functionalities in future types like a `Taxi`.
* **Interfaces** in Go exposes the behaviors of a type.

#### Memory Management

* No manual memory management, memory leak problems as in languages like C, C++.
* Go has a Garbage-Collector that will handle memory management for you.
* GC adds a bit overhead to execution time but reduces development effort significantly.



## Chapter-2 : Quick Start

Following things are demonstrated by a real-world go program to read data feeds from different sources and process them by search term.
[Source](https://github.com/goinaction/code/tree/master/chapter2/sample)

* **Productive** : minimal set of keywords, syntax, comprehensive built-in standard library
* **Performant** : access to low-level constructs

#### Main package

* `Imports` allows access to identifiers such as *types, functions, constants, interfaces*
* `package main` and `main()` are necessary for the compiler to build an executable program.
* `package`s indirects indentifiers declared inside and distinguishes other identifiers with same name in other `package`s
* Just like *Java*, package names must be same with folder path
* `_` , blank-identifier import will initialize (by calling `init()` function inside the package before identifiers are directly used
  * Go compiler does not let *empty* imports

```go
import (
	"os"
	_ "github.com/goinaction/code/chapter2/sample/matchers"
)
```

* All `init` functions in code files are called before `main` function.

#### Search package

#### 1) search.go

* `import` 

  * standard library only need to specify the *name* of the package.
  * non-standard libraries are searched by folder path inside `GOROOT` and `GOPATH` os environment variables.

* `identifiers`

  * package level variables : outside of any function
  * exported identifiers : starts with *Uppercase* letter e.g `Run()`
  * unexported/local identifiers : starts with *lowercase* letter

* `make()` : built-in function to create data-structures like *map, slices and channels*

* `map` : reference type that implements key-value data-structure

* all variables are initialized to their *zero-value*. `0` for numeric type, empty string for strings, `false` for booleans, `nil` for pointers

* `slice` : reference type that can implements dynamic array

* `function`s can return multiple values

  * common practice is to return a `value` and `error` value

  ```go
  // Retrieve the list of feeds to search through.
  feeds, err := RetrieveFeeds()
  	if err != nil {
  	log.Fatal(err)
  }
  ```

* `:=` : shorthand notation for *declare and assign* value to variable.

  * use `var` for declaration with default zero-values
  * use `:=` or short variable declaration for extra initialization

* when `main()` function exits, any `go routine`s are terminated too. Use *Wait Group* to wait for go routines to finish.

  * To reduce bugs and wasting resources, it is good practice to *manually end any go routines* running in background before the program exits.

* `for range` 

  * can be used to iterate *slices, arrays, strings, maps and channels*.
  * returns two values. First is index position and second is element.
  *  `_` blank-identifier can be used to ignore any unnecessary value from `for range` e.g *index*

* **map lookup** `myMap[key]` return two values. First is *element value* at key and Second is *boolean* if key exists or not.

* `go routines` accept annonymous functions that will be running independently.

  * by default, functions accept parameters by value (i.e original value is copied and passed)
  * use `pointers` to share *Mutatable State* between different functions or even go routines.

* go supports `closures` , inner annonymous functions have **direct-access** to the variables in outer scope.

#### 2) feed.go

* `struct`s can be used to define customer types
* fields in `struct`s can contain tags for additiona metadata

```go
// contains mapping metadata to read JSON objects
type Feed struct {
	Name string `json:"site"`
	URI string `json:"link"`
	Type string `json:"type"`
}
```

* `defer` : schedule a function call to execute after current function execution finishes.
  * works even if the function panics and terminates unexpectedly

```
func readSomeFile() {
	file, err := os.Open(file)
	// Schedule the file to be closed once
	// the function returns.
	defer file.Close()
	
	// code
}
```

#### 3) match.go/default.go

* `interface` defines behavior to be implemented via methods.

* `interface` name should have ***er*** suffix if `interface` has only one method. Use generalized name for multiple methods contained.

* `struct`s can be empty when they don't maintain state. Common when `struct` needs only to implement `interface`s.

* `receiver` in any function declaration bounds method calls to a specific receiver *type*.

  * use `pointer receiver`s when methods implemented needs to manipulate state of the *type* being used
    * Interface methods needs to be called with pointer types
  * use `value receiver`s when no state exists in creating *type*
    * interface methods can be called with both pointer and value types

  ```go
  // Method declared with a pointer receiver of type defaultMatcher
  func (m *defaultMatcher) Search(feed *Feed, searchTerm string)
  // Call the method via an interface type value
  var dm defaultMatcher
  var matcher Matcher = dm // Assign value to interface type
  matcher.Search(feed, "test") // Call interface method with value
  > go build
  cannot use dm (type defaultMatcher) as type Matcher in assignment
  // Method declared with a value receiver of type defaultMatcher
  func (m defaultMatcher) Search(feed *Feed, searchTerm string)
  // Call the method via an interface type value
  var dm defaultMatcher
  var matcher Matcher = &dm // Assign pointer to interface type
  matcher.Search(feed, "test") // Call interface method with pointer
  > go build
  Build Successful
  ```


### RSS matcher

* `encoding/xml` : builtin library to encode/decode **xml** files
* `net/http` : simple built-in library to make **HTTP** requests.
* `append` built-in function to add items (with dynamic resizing functionality)



### Summary

* **Packaging system** `Code file -> Package -> Folder`
* Use `pointers` to **share data** across *functions* and *goroutines*
* **Concurrency** -> *goroutines*, **Synchronization** -> *Channels*
* Go include many **built-in functions** to support data structures and daily programming requirements.
* **Interfaces** allows *generic* code and *frameworks*.



## Chapter 3 : Packaging and Tooling

> Tha idea is to separate semantic units of functionality into different packages. When you do this, you enable code reuse and control the use of the data inside each package. 



#### Packages

* small, reusable, single-functionality group of files organized into same directory
* All *.go* files must have package declared in first line of code
* Packages can only live in **single directory**
  * no multiple packages in same directory
  * single packages cannot be split across multiple directories
* For e.g, `net`  standard library is broken down into packages such as `http`, `cookiejar` , etc so that developers don't need to import the code that is not needed.

####Naming Convention

* *packages* should have the **SAME** name as *directory* name
* unique name is not necessary since *packages* are imported using full-path
* package name is used by default as import name but can be *overidden*.

#### Package main

* package `main` means the package can make executable binary file.
* contains the method `main()` from where the program starts running.
* executable package are a.k.a **commands**

#### Imports

* when import, compiler looks for packages in following locaions in order
  * Go installation directory (`/usr/local/go`)
  * `GOPATH` env variable path
* for developing that will be shared as libraries/packages, put them under `GOPATH/src`
* Imported packages are required use in code otherwise compiler will give an error
  * this eliminates unused, bloated imports problem that can be found in other languages

#### Remote Imports

* supports tools to fetch code from distributed VCS (GitHub, Bitbucket, etc)

* `go get` fetches code over network and puts them inside `GOPATH` so compiler can find them when building dependencies.

  ```go
  import "github.com/spf13/viper"
  ```

#### Named Imports

* *Named imports* to avoid import duplication when packages have same name.

  ```go
  import (
  	"fmt"
  	myfmt "mylib/fmt"
  )
  ```

#### Go Tools

* **go build** : compiles a .go file and creates executeble binary file (cross-platform)
* **go get** : get a remote package from a given URL (e.g GitHub)
* **go run** : executes the compiled executable binary
* **go vet** : checks basic common errors like lacking function parameter, method signature error ,etc
* **go fmt** : formats the go code to predetermined layout
* **go doc** : grabs and shows the documentation of a package 
  * or use  `godoc -http=6060` command to browse the whole documentation repo in web browser at `localhost:6060`
  * Can also include your code Comments into `go doc` documentation if you follow certain convention

#### Dependency Management

> Built-in go tools does not have proper support managing *different versions* of packages and *reproducible builds* because Go was mainly designed for code sharing and expects everyone to share the latest version.

* **godep** and **vendor** are tools that wraps go tooling that uses technique like *vendoring* and *import-path-rewriting* that follows steps as below

  * copy all third-party/vendor dependencies into project directory
  * rewrite import path

* **Gb ** : provides reproducible build tool without wrapping Go tools or `GOPATH`

  * project based approach
  * all versioned dependencies are put under `/vendor` path inside project root directory
  * Gb tool looks for packages in import firstly under `/vendor/src`  if they are not found under the `/src` subdirectories.
  * not compatible with Go tooling and needs separate command for builds

  ```bash
  gb build all
  ```

### Summary

* Packages are the basic unit of code organization in Go.
* Your GOPATH determines on disk where Go source code is saved, compiled, and
  installed.
* You can set your GOPATH for each different project, keeping all of your source
  and dependencies separate 
* You can use packages created by other people by using go get to fetch and
  install them in your GOPATH.
* Go was designed with code sharing as a central driving feature of the language.
  * It’s recommended that you use vendoring to manage dependencies.
  * There are several community-developed tools for dependency management
    such as godep, vendor, and gb.



## Chapter 4 : Arrays, Slices and Maps



### 1) Array internals and fundamentals

* basic data structure that supports *Slices and Maps*
* **fixed** length, **continuous** block of items of **same** type
* initialized to item's zero-value when being declared
* array's *type or length* cannot be changed after declaration

```go
// Declare an integer array of five elements.
// Initialize each element with a specific value.
array := [5]int{10, 20, 30, 40, 50}

// Declare an integer array.
// Initialize each element with a specific value.
// Capacity is determined based on the number of values initialized.
array := [...]int{10, 20, 30, 40, 50}

// Declare an integer array of five elements.
// Initialize index 1 and 2 with specific values.
// The rest of the elements contain their zero value.
array := [5]int{1: 10, 2: 20}
```

* **arrays are values** in Go

  * when you assign an array to other array, you get two arrays with two copies of the same value

  ```go
  // Declare a string array of five elements.
  var array1 [5]string
  // Declare a second string array of five elements.
  // Initialize the array with colors.
  array2 := [5]string{"Red", "Blue", "Green", "Yellow", "Pink"}
  // Copy the values from array2 into array1.
  array1 = array2
  ```

  * use *pointers* to share values inside arrays (for example, passing arrays between *functions* or *Goroutines*) for better performance

  ```go
  // Allocate a LARGE array of 8 megabytes.
  var array [1e6]int
  // Pass the address of the array to the function foo.
  foo(&array)
  // Function foo accepts a pointer to an array of one million integers.
  func foo(array *[1e6]int) {
  ...
  }
  ```



### 2) Slices internals and fundamentals

> basic data structure to handle collections of data



#### Internals

* Tiny objects that *abstracts* and *manipulates* *underlying array*
* **Dynamic** arrays that can **expand** or **shrink**
* Slices uses *three* *metadata* to manipulate arrays
  * **Pointer** to array (i.e memory location)
  * **Size** : length of array that slice can have access
  * **Capacity** : length of underlying array
* Since **slices use pointers**, they *share the array data value in memory* when *creating* new slices  or *appending* the original slice



####Making slices

* `make` built-in function

```go
// Create a slice of strings.
// Contains a length and capacity of 5 elements.
slice := make([]string, 5)

// Create a slice of integers.
// Contains a length of 3 and has a capacity of 5 elements.
slice := make([]int, 3, 5)
```

* **slice literal**

```go
// Create a slice of strings.
// Contains a length and capacity of 5 elements.
slice := []string{"Red", "Blue", "Green", "Yellow", "Pink"}
```

* `nil` slice (useful when returning error case from a function that returns a slice)

```
// Create a nil slice of integers.
var slice []int
```

* **empty slice**

```go
// Use make to create an empty slice of integers.
slice := make([]int, 0)
// Use a slice literal to create an empty slice of integers.
slice := []int{}
```



#### Adding elements to slices

* slices can be appended using built-in function `append()`
  * `append` can accept multiple values, or even use `...` operator to append elements inside another slice

```go
// Create a slice of integers.
// Contains a length and capacity of 5 elements.
slice := []int{10, 20, 30, 40, 50}
// Create a new slice.
// Contains a length of 2 and capacity of 4 elements.
newSlice := slice[1:3]
// Allocate a new element from capacity.
// Assign the value of 60 to the new element.
newSlice = append(newSlice, 60)

// resulting array
slice
[10, 20, 30 60, 50]
```

* Appending on `newSlice` will affect *index3* of original `slice` since they share the same array under the hood.
* If underlying array has no more capacity left, `append()` function will create a new array and copy the values
  * `append()` function is clever that *capacity is doubled* for under 1,000 elements array. 1.25%, 25% for larger arrays
* **[WARNING]** It is easy to get lost what slices use which underlying array and suddenly changes are affected to other slices. Use **three-index-slices** with same length and capacity.

```go
// Create a slice of strings.
// Contains a length and capacity of 5 elements.
source := []string{"Apple", "Orange", "Plum", "Banana", "Grape"}
// Slice the third element and restrict the capacity.
// Contains a length and capacity of 1 element.
slice := source[2:3:3]
// Append a new string to the slice.
slice = append(slice, "Kiwi")
```

* appending to `slice` in above, will create a new array since its capacity is not enough thus avoid changes to underlying array that `source` is also referenced.



#### Iterating slices

* `for range` can be used to iterate over slices

```go
// Create a slice of integers.
// Contains a length and capacity of 4 elements.
slice := []int{10, 20, 30, 40}
// Iterate over each element and display each value.
for index, value := range slice {
fmt.Printf("Index: %d Value: %d\n", index, value)
}
```

* **[Warning]** Since `for range` copy values from original slice, if you use pointers to make changes to underlying array, only the copy value will be affected.

```go
// Create a slice of integers.
// Contains a length and capacity of 4 elements.
slice := []int{10, 20, 30, 40}
// Iterate over each element and display the value and addresses.
for index, value := range slice {
fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n",
value, &value, &slice[index])
}
Output:
Value: 10 Value-Addr: 10500168 ElemAddr: 1052E100
Value: 20 Value-Addr: 10500168 ElemAddr: 1052E104
Value: 30 Value-Addr: 10500168 ElemAddr: 1052E108
Value: 40 Value-Addr: 10500168 ElemAddr: 1052E10C
// the address for the value variable is always the same because it’s a variable that contains a copy
```



#### Passing slices between functions

* *very cheap* since slice contains only metada ( array address, size integer, etc) even if value is copied to pass.

- *On a 64-bit architecture, a slice requires 24 bytes of memory. The pointer field*
  *requires 8 bytes, and the length and capacity fields require 8 bytes respectively.*

```go
// Allocate a slice of 1 million integers.
slice := make([]int, 1e6)
// Pass the slice to the function foo.
slice = foo(slice)
// Function foo accepts a slice of integers and returns the slice back.
func foo(slice []int) []int {
...
return slice
}
```



### 3) Maps internals and fundamentals

> **unordered*** collection of ***key/value*** pairs



####Internals

* implemented using ***hash table***

  * there are two main parts in a map
    * buckets
    * data
  * map entries are grouped into ***buckets*** (*an array of hashes*)
  * *bucket location* or *index* or *hash* is determined by passing the *entry key* to the *hash-function* (*a hash contains 16 bit numeric value*)
  * the better the *hash-function*, the more evenly it will distribute the *entries* across the *buckets*, the quicker you can find *entries*, the better the performance of the *map*


  ![go-maps](https://www.ardanlabs.com/images/goinggo/Screen+Shot+2013-12-31+at+7.01.15+PM.png)

  * Lower order bit (LOB) is used to select the hash (*LOB is mostly the lower-half  (8 bit binary) of the 16 bit numeric that was generated from key*)
  * As in figure, for data part of the map, there are two sub data structures
    * an array of *top eight Higher Order Bits (HOB)*, that separate entries in each respective bucket
    * an array of bytes that contains stores key/value pairs for the respective bucket



####Manipulation

* *maps* can be created by using built-in `make()` function or using *map literals*

  * using map literals is idiomatic way of map creation
  * map *keys* can be any *primitive value* or *struct type* as long as it can be compared with `==` operator
  * *slices, functions* and *struct* types that contain slices cannot be used as keys and will produce *compile error*
  * meanwhile, *map value* can be of any type including *slices*

  ```go
  // Create a map with a key of type string and a value of type int.
  dict := make(map[string]int)
  // Create a map with a key and value of type string.
  // Initialize the map with 2 key/value pairs.
  dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}
  ```

* **assigning** values to a map

  ```go
  // Create an empty map to store colors and their color codes.
  colors := map[string]string{}
  // Add the Red color code to the map.
  colors["Red"] = "#da1337"
  ```

* can receive two return values when **retrieving** from map

  - should always check if a map value exists

  ```go
  // Retrieve the value for the key "Blue".
  value, exists := colors["Blue"]
  // Did this key exist?
  if exists {
  	fmt.Println(value)
  }
  ```

* **Iterating and removing**

  ```go
  // Create a map of colors and color hex codes.
  colors := map[string]string{
      "AliceBlue": "#f0f8ff",
      "Coral": "#ff7F50",
      "DarkGray": "#a9a9a9",
      "ForestGreen": "#228b22",
  }
  // Display all the colors in the map.
  for key, value := range colors {
  	fmt.Printf("Key: %s Value: %s\n", key, value)
  }
  
  // Remove the key/value pair for the key "Coral".
  delete(colors, "Coral")
  ```

* `nil` maps

  * can be useful when returning error case from a function

  ```go
  // Create a nil map by just declaring the map.
  var colors map[string]string
  // Add the Red color code to the map.
  colors["Red"] = "#da1337"
  
  Runtime Error:
  panic: runtime error: assignment to entry in nil map
  ```



####Passing maps between functions

* Maps are designed to be cheap, similar to slices. That means, maps **are not copied when passing to functions.**
* Modifying a map will reflect in all occurances to the map i.e in all functions that received and codes that are using the map



### Summary

* Arrays are the building blocks for both slices and maps.
* Slices are the idiomatic way in Go you work with collections of data. Maps are
  the way you work with key/value pairs of data.
* The built-in function `make` allows you to create slices and maps with initial
  length and capacity. Slice and map literals can be used as well and support setting
  initial values for use.
* Slices have a capacity restriction, but can be extended using the built-in function
  `append`. Maps don’t have a capacity or any restriction on growth.
* Through the use of composition, you can create multidimensional arrays and
  slices. You can also create maps with values that are slices and other maps. A
  slice can’t be used as a map key.
* Passing a slice or map to a function is cheap and doesn’t make a copy of the
  underlying data structure.



## Chapter 5 : Go's Type System

> Go is statically typed language. Compiler wants to know the type of every piece of value in the program. Knowing the type information ahead of time ensures to,
>
> *  reduce bugs of using the unsupported operations of a type in codes (will be warned in compile time)
> *  Type declarations as auto-checked documentation for code and enables predictable, understandable code
> *  as opposite of dynamically types languages, the runtime is more efficient since there is no type guessing 
> *  Go also supports short-hand initialization which reduces the tedius work of type-casting when declaring new variables or receiving return values from functions