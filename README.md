# sector

[![License MIT](https://img.shields.io/badge/License-MIT-blue.svg)](http://opensource.org/licenses/MIT) [![GoDoc](https://godoc.org/github.com/dc0d/sector?status.svg)](http://godoc.org/github.com/dc0d/sector) [![Go Report Card](https://goreportcard.com/badge/github.com/dc0d/sector)](https://goreportcard.com/report/github.com/dc0d/sector) [![Build Status](https://travis-ci.org/dc0d/sector.svg?branch=master)](http://travis-ci.org/dc0d/sector) [![codecov](https://codecov.io/gh/dc0d/sector/branch/master/graph/badge.svg)](https://codecov.io/gh/dc0d/sector)

*sector* - for Simple Injector - provides a Dependency Injection mechanism for Go.

Put it simply, _sector_ fills pointers with values come from _factories_. So we have the *factory* - the constructor (role) - and the *injector*.

A *factory* implements the `Factory` interface. In it's simplest form, it's just a function with this signature, `func(interface{}) bool` and uses the Go's _type switch_ to fill the pointers:

```go
func myFactory(ptr interface{}) bool {
	switch x := ptr.(type) {
	case *int:
		*x = 1001
	case *string:
		*x = `injected string`
	case *Trait:
		buffer := Trait{}
		*x = buffer
	case *Data:
		buffer := Data{}
		*x = buffer
	case *map[string]interface{}:
		*x = make(map[string]interface{})
		(*x)[`asset`] = `another injected string`
	case *SI:
		v := SIO{`yet another injected string`}
		*x = &v
	case *[]int:
		*x = []int{1, 2, 3}
	default:
                // this factory does not fill pointers of this type
		return false
	}

	return true
}
```

Then we use this factory in our _injector_ to fill in the fields of a struct.

```go
dj := NewInjector(FactoryFunc(genericFactory))
```

Now we use this injector to inject desired values into struct's fields.

```go
var s Sample
dj.Inject(&s)
```

Fields that are tagged with `inject:"+"` will get filled with proper value from the _factory_. Fields tagged with `inject:"*"` will get filled recursively down the tree.

```go
type Sample struct {
	Trait `inject:"*"`      // inject field, recursively, all down the data tree  

	N  int    `inject:"+"`  // inject field
	S  string `inject:"+"`
	D  Data   `inject:"*"`
	NL []int  `inject:"+"`

	Map     map[string]interface{} `inject:"+"`
	DataPtr *Data                  `inject:"*"`

	SI SI `inject:"+"`
}
```
It is also possible to inject arguments of a function using the `Invoke` method.
