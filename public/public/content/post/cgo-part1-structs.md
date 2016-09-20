+++
date = "2016-04-20T12:00:00"
draft = false
image = ""
tags = ["golang", "c", "cgo"]
title = "Best Practices in cgo Part I: Structs"
math = false
+++

One of the libraries that makes Go so powerful is cgo, a set of bindings for calling C code from Go.  This is called a Foreign Function Interface (or FFI) and despite the versatility it adds to Go, it can be a tricky library to use.  To help ease new users through some of the trickier aspects of cgo, here are some general best practices.

## Structs
Handing C structs is one of the most common issues with cgo code.  Take a look at the code down below:
```go
package person

import (
	"fmt"
)

// #include "../structs.h"
import "C"

func PrintPerson() {
	p := GetPerson()
	fmt.Println("name:", C.GoString(p.name))
	fmt.Println("age:", p.age)
}

func GetPerson() *C.Person {
	p := C.new_person()
	p.name = C.CString("John Doe")
	p.age = 39
	return p
}

```
If your code looks something like this, then you shouldn't have a problem.  But let's say we import this library in another project and try to call foo from there:
```go
package employee

import (
	"fmt"

	"github.com/14rcole/cgo-examples/structs/person"
)

// #include "../structs.h"
import "C"

type employee struct {
	name    string
	age     int
	title   string
	company string
}

func PrintEmployee() {
	e := GetEmployee(person.GetPerson())
	fmt.Println("name:", e.name)
	fmt.Println("age:", e.age)
	fmt.Println("title:", e.title)
	fmt.Println("company:", e.company)
}

func GetEmployee(p *C.Person) employee {
	var e employee
	e.name = C.GoString(p.name)
	e.age = (int)(p.age)
	e.title = "Software Engineer"
	e.company = "Some Cool Company, Inc."

	return e
}
```
When you run this code, you should receive an error that looks something like this:
```
$ structs/employee/employee.go:19: cannot use person.GetPerson() (type *person.C.struct_Person) as type *C.struct_Person in argument to GetEmployee
```
This is because when person.GetPerson() calls new_person in structs.h, it receives a C struct.  This is assigned to cgos concept of that C struct, which is _C.struct_Person_.  However, when being passed between functions, the struct has the package name associated with it, just like other Go functions.  So it becomes _person.C.struct_Person_ which, according to Go's type system, is not equivalent to _C.struct_Person_

#### The Solution
So how to we get around this?  We can't use a typedef.  When the Go compiler reads `type foo bar` it sees foo as a new type that is identical to type bar, not simply as an alias for it.   So there are two valid solutions to this.  The first is to create a struct that only contains a pointer
```go
package person

import (
	"fmt"
)

// #include "../structs.h"
import "C"

type CPerson struct {
    ptr unsafe.Pointer
}

func (cp CPerson) CPersonToNative() *C.Person {
    return cp.ptr
}

func NewCPerson(p unsafe.Pointer) CPerson {
    var cp CPerson
    cp.ptr = p
    return CPerson
}

func PrintPerson() {
	p := GetPerson()
	fmt.Println("name:", C.GoString(p.name))
	fmt.Println("age:", p.age)
}

func GetPerson() CPerson {
	p := NewCPerson(unsafe.Pointer(C.new_person()))
	p.name = C.CString("John Doe")
	p.age = 39
	return p
}
```
After this, anytime the C struct Person needs to be used outside of the Go struct, package.CPerson can be used instead.  CPerson.CPersonToNative() can be used to generate a native C struct to pass back into a C function.  Of course, that can be painful, as you'll begin to have nested function calls on function calls and typecasts for that unsafe.Pointer that's being passed around.  Even something as simple as accessing a string value in the struct would be as complex as this:
```go
var name := C.GoString(((*C.Person)(cp.CPersonToNative())).name)
```
Pretty messy, huh?  There is another way to handle structs as well.  It takes more work to implement and requires more computation to convert between the C and Go versions of the struct but it makes accessing values in the struct far easier.  You could unmarshal all of the data from your C struct into a Go struct.  Converting back to native would require remarshalling into a C struct.  That would look something like this:
```go
package person

import (
	"fmt"
)

// #include "../structs.h"
import "C"

type CPerson struct {
    ptr unsafe.Pointer
}

func (cp CPerson) CPersonToNative() *C.Person {
    p *C.person
    p.name = C.CString(cp.name)
    p.age = (C.int)(cp.age)
    return p
}

func NewCPerson(ptr unsafe.Pointer) CPerson {
    var cp CPerson
    p := (*C.Person)(ptr)
    cp.name = C.GoString(p.name)
    cp.age = (int)(p.age)
    return cp
}

func PrintPerson() {
	p := GetPerson()
	fmt.Println("name:", C.GoString(p.name))
	fmt.Println("age:", p.age)
}

func GetPerson() CPerson {
	p := NewCPerson(unsafe.Pointer(C.new_person()))
	p.name = C.CString("John Doe")
	p.age = 39
	return p
}
```
Altough the unmarshalling here was straightforward enough, I'm sure you can imagine how complex it could all get, especially if the C structs contained other C structs.  This may not always be a viable option, but it certainly helps with accessing variables.

## Summary
So that's everything.  A best practices guide for C structs in Go.  Unfortunately there's no completely elegant solution to the problem.  FFI stuff is hard.  That being said, hopefully this guide and the next installments will get you started in the right direction the next time you have to type `import "C"`
