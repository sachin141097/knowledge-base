## What is an Interface in Go?
An interface type in Go is kind of like definition. It defines and describes the exact methods some other types must have

One example of interface type from the standard library is `fmt.Stringer` interface which looks like this

```go
type Stringer interface{
	String() string
}
```

We say that something satisfies the interface (or implements this interface) if it has method with exact same signature `String() string`

For example,the following `Book` type satisfies the interface because it has `String() string` method

```go
type Book struct{
	Title string
	Author string
}

func (b Book) String() string{
	return fmt.Sprintf("Book: %s - %s", b.Title, b.Author)
}
```

The following `Count` type also satisfies the `fmt.Stringer` interface

```go
type Count int
func (c Count)String() string{
	return strconv.Itoa(int(c))
}
```

Wherever you see declaration in Go (such as variable,function parameter or struct field)which has an interface type,you can use object of any type as long as it satisfies the interface

For example let's say you have the following function
```go
func WriteLog(s fmt.Stringer){
	log.Print(s.String())
}
```

Because the `WriteLog` function uses the `fmt.Stringer` interface type in its parameter declaration,we can pass any object that satisfies the `fmt.Stringer` interface. For example we could pass either of `Book` or `Count` types to `WriteLog` method

```go
package main

import (
	"fmt"
	"log"
	"strconv"
)

type Book struct {
	Title  string
	Author string
}

func (b Book) String() string {
	return fmt.Sprintf("Book:%s-%s", b.Title, b.Author)
}

type Count int

func (c Count) String() string {
	return strconv.Itoa(int(c))
}

func WriteLog(s fmt.Stringer) {
	log.Print(s.String())

}
func main() {
	book := Book{"And then there were none", "Agatha Christie"}
	WriteLog(book)
	count := Count(3)
	WriteLog(count)

}

```

If you run the code you will get output like this

```bash
2026/04/30 19:30:43 Book:And then there were none-Agatha Christie
2026/04/30 19:30:43 3
```

## Why are they useful?

1. To help reduce duplication or boilerplate code
2. To make it easier to use mocks instead of real objects in unit tests
3. Helps enforce decoupling between parts of your codebase

### Reducing boilerplate code
Imagine having `Customer` struct containing some data about customer. In one part of our codebase we want to write customer information to `bytes.Buffer` and in another part of codebase we want to write the customer information to an `os.File` on disk

This is the scenario we can use go interfaces to reduce boilerplate code

The first thing you need to know is that Go has `io.Writer` interface type which looks like this

```go
type Writer interface{
	Write(p []byte)(n int,err error)
}
```

And we can leverage the fact that both `bytes.Buffer` and `os.File` satisfy this interface,due to them having `bytes.Buffer.Write()` and `os.File.Write()` method respectively


```go
package main

import (
	"bytes"
	"encoding/json"
	"io"
	"log"
	"os"
)

type Customer struct {
	Name string
	Age  int
}

func (c *Customer) WriteJSON(w io.Writer) error {
	js, err := json.Marshal(c)
	if err != nil {
		return err
	}
	_, err = w.Write(js)
	return err
}

func main() {
	c := &Customer{Name: "Alice", Age: 21}
	var buf bytes.Buffer
	err := c.WriteJSON(&buf)
	if err != nil {
		log.Fatal(err)
	}
	f, err := os.Create("/tmp/customer")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()
	err = c.WriteJSON(f)
	if err != nil {
		log.Fatal(err)
	}

}

```

### Unit testing and mocking
Let's say you run a shop,and you store information about number of customers and sales in a PostgreSQL database. You want to write some code that calculates sales rate for the past 24 hours

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"time"
)

type ShopDB struct {
	*sql.DB
}

func (sdb *ShopDB) CountCustomers(since time.Time) (int, error) {
	var count int
	err := sdb.QueryRow("SELECT count(*) FROM customers WHERE timestamp > $1", since).Scan(&count)
	return count, err
}
func (sdb *ShopDB) CountSales(since time.Time) (int, error) {
	var count int
	err := sdb.QueryRow("SELECT count(*) FROM sales WHERE timestamp > $1", since).Scan(&count)
	return count, err
}
func main() {
	db, err := sql.Open("postgres", "postgres://user:pass@localhost/db")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	shopDB := &ShopDB{db}
	sr, err := calculateSalesRate(shopDB)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf(sr)

}
func calculateSalesRate(sdb *ShopDB) (string, error) {
	since := time.Now().Add(-24 * time.Hour)

	sales, err := sdb.CountSales(since)
	if err != nil {
		return "", err
	}

	customers, err := sdb.CountCustomers(since)
	if err != nil {
		return "", err
	}

	rate := float64(sales) / float64(customers)
	return fmt.Sprintf("%.2f", rate), nil
}

```

What if we want to create a unit test for the `calculateSalesRate()` function to make sure that math logic is working correctly

We would need test instance of our postgresql database along with setup and teardown scripts to scaffold the database with dummy data. That's quite lot of work

A solution is to create our own interface type which describes `CountSales()` and `CountCustomers()` methods that the `calculateSalesRate()` function relies on

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"time"
)

type ShopModel interface {
	CountCustomers(time.Time) (int, error)
	CountSales(time.Time) (int, error)
}
type ShopDB struct {
	*sql.DB
}

func (sdb *ShopDB) CountCustomers(since time.Time) (int, error) {
	var count int
	err := sdb.QueryRow("SELECT count(*) FROM customers WHERE timestamp > $1", since).Scan(&count)
	return count, err
}

func (sdb *ShopDB) CountSales(since time.Time) (int, error) {
	var count int
	err := sdb.QueryRow("SELECT count(*) FROM sales WHERE timestamp > $1", since).Scan(&count)
	return count, err
}
func main() {
	db, err := sql.Open("postgres", "postgres://user:pass@localhost/db")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	shopDB := &ShopDB{db}
	sr, err := calculateSalesRate(shopDB)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf(sr)
}

// Swap this to use the ShopModel interface type as the parameter, instead of the
// concrete *ShopDB type.
func calculateSalesRate(sm ShopModel) (string, error) {
	since := time.Now().Add(-24 * time.Hour)

	sales, err := sm.CountSales(since)
	if err != nil {
		return "", err
	}

	customers, err := sm.CountCustomers(since)
	if err != nil {
		return "", err
	}

	rate := float64(sales) / float64(customers)
	return fmt.Sprintf("%.2f", rate), nil
}

```

With that it is straightforward for us to create a mock which satisfies our `ShopModel` interface

```go
package main

import (
    "testing"
    "time"
)

type MockShopDB struct{}

func (m *MockShopDB) CountCustomers(_ time.Time) (int, error) {
    return 1000, nil
}

func (m *MockShopDB) CountSales(_ time.Time) (int, error) {
    return 333, nil
}

func TestCalculateSalesRate(t *testing.T) {
    // Initialize the mock.
    m := &MockShopDB{}
    // Pass the mock to the calculateSalesRate() function.
    sr, err := calculateSalesRate(m)
    if err != nil {
        t.Fatal(err)
    }

    // Check that the return value is as expected, based on the mocked
    // inputs.
    exp := "0.33"
    if sr != exp {
        t.Fatalf("got %v; expected %v", sr, exp)
    }
}
```

### What is the empty interface?
The empty interface type essentially describes no methods. It has no rules and because of that any and every object satisfies empty interface

The empty interface type `interface{}` is kind of like wildcard. Wherever you see it in declaration you can use object of any type

```go
package main

import "fmt"

func main() {
    person := make(map[string]interface{}, 0)

    person["name"] = "Alice"
    person["age"] = 21
    person["height"] = 167.64

    fmt.Printf("%+v", person)
}
```

But there's an important thing to point out when it comes to retrieving and using a value from this map.

For example, let's say that we want to get the `"age"` value and increment it by 1. If you write something like the following code, it will fail to compile:

```go
package main

import "log"

func main() {
    person := make(map[string]interface{}, 0)

    person["name"] = "Alice"
    person["age"] = 21
    person["height"] = 167.64

    person["age"] = person["age"] + 1

    fmt.Printf("%+v", person)
}
```


And you'll get the following error message:

```bash
invalid operation: person["age"] + 1 (mismatched types interface {} and int)
```

This happens because the value stored in the map takes on the type `interface{}`, and ceases to have it's original, underlying, type of `int`. Because it's no longer an `int` type we cannot add 1 to it.

To get around this this, you need to type assert the value back to an `int` before using it. Like so:

```go
package main

import "log"

func main() {
    person := make(map[string]interface{}, 0)

    person["name"] = "Alice"
    person["age"] = 21
    person["height"] = 167.64

    age, ok := person["age"].(int)
    if !ok {
        log.Fatal("could not assert value to int")
        return
    }

    person["age"] = age + 1

    log.Printf("%+v", person)
}
```

So when should you use the empty interface type in your own code?

The answer is _probably not that often_. If you find yourself reaching for it, pause and consider whether using `interface{}` is really the right option. As a general rule it's clearer, safer and more performant to use concrete types — or non-empty interface types — instead. In the code snippet above, it would have been more appropriate to define a `Person` struct with relevant typed fields similar to this:

```go
type Person struct {
    Name   string
    Age    int
    Height float32
}
```

But that said, the empty interface is useful in situations where you need to accept and work with unpredictable or user-defined types. You'll see it used in a number of places throughout the standard library for that exact reason, such as in the [`gob.Encode`](https://pkg.go.dev/encoding/gob/#Encoder.Encode), [`fmt.Print`](https://pkg.go.dev/fmt/#Print) and [`template.Execute`](https://pkg.go.dev/text/template/#Template.Execute) functions.

### The any identifier
Go 1.18 introduced a new [predeclared identifier](https://tip.golang.org/ref/spec#Predeclared_identifiers) called [`any`](https://pkg.go.dev/builtin#any), which is an alias for the empty interface `interface{}`,

The `any` identifier is straight-up syntactic sugar – using it in your code is equivalent in all ways to using `interface{}`– it means exactly the same thing and has exactly the same behavior. So writing `map[string]any` in your code is exactly the same as writing `map[string]interface{}` in terms of it's behavior.

In most modern Go codebases, you'll normally see `any` being used rather than `interface{}`. This is simply because it's shorter and saves typing, and more clearly conveys to the reader that you can use _any type_ here.