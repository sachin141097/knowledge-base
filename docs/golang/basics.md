- [Struct](#struct)
	- [Struct Literals](#struct-literals)



## Struct
A struct is an aggregate data type that groups together zero or more named values of arbitary types as a single entity. Each value is called field. The classic example of struct from data processing is the employee record,whose field are unique ID,the employee's name,address,date of birth,position,salary,manager and the like. All of these fields are collected into single entity that can be copied as a unit,passed to functions and returned by them.Stored in an arrays and so on.

```go
type Employee struct{
	ID int
	Name string
	Address string
	DoB time.Time
	Position string
	Salary int
	ManagerID int
}
var dilbert Employee
```
The individual fields of dilbert are accessed using dot notation like `dilbert.Name` and `dilbert.DoB` 
```go
dilbert.Salary-=5000
```

>The name of the struct field is exported if it begins with a capital letter;this is Go's main access control mechanism. A struct type may contain mixture of exported and unexported fields

A named struct type S can't declare a field of same type S: an aggregate value cannot contain itself. But S may declare a field of the Pointer type `*S` which let us create recursive data structures like linked lists and trees

The zero value for a struct is composed of the zero values of each of its fields.

### Struct Literals
A value of struct type can be written using a struct literal that specifies value for its fields

```go
type Point struct {X,Y int}
p:=Point{1,2}
```
There are two forms of struct literal. The first form shown above,requires that a value be specified for every field,in the right order. It burdens the writer (and reader) with remembering exactly what the fields are,and it makes the code fragile should the set of fields later grow or be reordered.

More often the second form is used,in which a struct value is initialized by listing some or all of the field names and their corresponding values
```go
anim:=gif.GIF{LoopCount:nframes}
```
If a field is omitted in this kind of literal,it is set to zero value for its type. Because names are provided the order of the fields doesn't matter.

The two forms cannot be mixed in the same literal.Nor can you use the (order-based) first form of literal to sneak around the rule that unexported identifiers may not be referred to from another package

```go
package p
type T struct{a,b int}// a and b are not exported

package q
import "p"
var __ = p.T{a:1,b:2}//compile error: can't reference a,b
var __ = p.T{1,2}// compile error: can't reference a,b
```
Struct values can be passed as arguments to functions and returned from them. For instance,this functions scales a point by a specified factor

```go
func Scale(p Point,factor int)Point{
	return Point{p.X*factor,p.Y*factor}
}


fmt.Println(Scale(Point{1,2},5))//"{5 10}"
```

>For efficiency larger struct types are usually passed to or returned from functions indirectly using a pointer, and this is required if the function must modify its argument,since in a call by value language like Go,the called function receives only a copy of an argument,not a reference to the original argument

```go
func Bonus(e *Employee,percent int)int{
	return e.Salary*percent/100
}
```

Because structs are so commonly dealt with through pointers,it is possible to use this shorthand notation to create and initialize a struct variable and obtain its address

```go
pp:=&Point{1,2}
```
It is exactly equivalent to

```go
pp:=new(Point)
*pp=Point{1,2}
```

