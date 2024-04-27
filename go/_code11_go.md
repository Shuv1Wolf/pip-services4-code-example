#### MethodReflector

```
// Method Reflector

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

type ObjectA struct {
}

func (c *ObjectA) MethodA() int {
	return 123
}

func (c *ObjectA) MethodB() {
	fmt.Println("hello world b")
}

func main() {
	myObjectA := &ObjectA{}

	// Obtain all methods in ObjectA
	methods1 := creflect.MethodReflector.GetMethodNames(myObjectA)
	fmt.Println("The methods in myObjectA are: ", methods1)

	// Ask whether a specific method exists or not
	methods2 := creflect.MethodReflector.HasMethod(myObjectA, "methodA")
	fmt.Println("methodA belongs to myObjectA: ", methods2)

	methods3 := creflect.MethodReflector.HasMethod(myObjectA, "methodC")
	fmt.Println("methodC belongs to myObjectA: ", methods3)

	// Invoke a method in ObjectA
	methods4 := creflect.MethodReflector.InvokeMethod(myObjectA, "methodA")
	fmt.Println("After running methodA the result is: ", methods4)

}
```

#### b) ObjectReader

```
// Object reader

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

type ObjectA struct {
	Param1 string
	Param2 int
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
	}
}

func (c *ObjectA) MethodA() int {
	return 123
}

func main() {
	myObjectA := NewObjectA()

	// Obtain all properties in ObjectA
	properties := creflect.ObjectReader.GetPropertyNames(myObjectA)
	fmt.Println("The properties in myObjectA are: ", properties)

	// Obtain the value of a property in myObjectA
	value1 := creflect.ObjectReader.GetProperty(myObjectA, "Param1")
	fmt.Println("The value of Param1 is: ", value1)

	value2 := creflect.ObjectReader.GetProperties(myObjectA)
	fmt.Println("The properties and values in myObjectA are: ", value2)
}
```

```
// Obtain properties from a map (dictionary)
myMap := map[string]interface{}{"key1": 123, "key2": "ABC"}

hasProperty1 := creflect.ObjectReader.HasProperty(myMap, "key1")
value1 := creflect.ObjectReader.GetProperty(myMap, "key1")
fmt.Println("MyMap contains key1: ", hasProperty1)
fmt.Println("The value of key1 is : ", value1)

// Obtain properties from an array
myArray := []int{1, 2, 3}
hasProperty2 := creflect.ObjectReader.HasProperty(myArray, "5")
hasProperty3 := creflect.ObjectReader.HasProperty(myArray, "0")
value2 := creflect.ObjectReader.GetProperty(myArray, "0")

fmt.Println("myArray contains an element with index 5: ", hasProperty2)
fmt.Println("myArray contains an element with index 0: ", hasProperty3)
fmt.Println("The value stored at postion 0 is: ", value2)
```

#### ObjectWriter

```
// Object writer - Setting property values

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	Param1 string
	Param2 int
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
	}
}

func (c *ObjectA) MethodA() int {
	return 123
}

func main() {
	myObjectA := NewObjectA()

	value1 := creflect.ObjectReader.GetProperty(myObjectA, "Param1")
	fmt.Println("The value of param1 is: ", value1)

	creflect.ObjectWriter.SetProperty(myObjectA, "Param1", "hello 2")
	value2 := creflect.ObjectReader.GetProperty(myObjectA, "Param1")
	fmt.Println("The new value of param1 is: ", value2)

	// Map (dictionary)
	myMap := map[string]interface{}{"key1": 123, "key2": "ABC"}
	creflect.ObjectWriter.SetProperties(myMap, map[string]interface{}{"key1": 15422, "key2": "ab"})
	value4 := creflect.ObjectReader.GetProperties(myMap)
	fmt.Println("The new values in the map are : ", value4)

	myMap = map[string]interface{}{"key1": 123, "key2": "ABC"}
	creflect.ObjectWriter.SetProperty(myMap, "key1", "XYZ")
	value2 = creflect.ObjectReader.GetProperty(myMap, "key1")
	fmt.Println("The new value in the map is : ", value2)

	// Array
	myArray := []int{1, 2, 3}
	creflect.ObjectWriter.SetProperty(myArray, "0", 123)
	value3 := creflect.ObjectReader.GetProperty(myArray, "0")
	fmt.Println("The new value in the array is : ", value3)
}

```

#### PropertyReflector

```
// Property reflector

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	Param1 string
	Param2 int
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
	}
}

func (c *ObjectA) MethodA() int {
	return 123
}

func main() {
	myObjectA := NewObjectA()

	// Obtain all property names
	properties := creflect.PropertyReflector.GetPropertyNames(myObjectA)
	fmt.Println("The properties of myObjectA are: ", properties)

	// Find out whether an object has a property or not
	hasParam1 := creflect.PropertyReflector.HasProperty(myObjectA, "Param1")
	fmt.Println("ClassA contains param1: ", hasParam1)

	// Obtain all property names and their values
	value3 := creflect.PropertyReflector.GetProperties(myObjectA)
	fmt.Println("The properties of myObjectA are: ", value3)

	// Change the value of a parameter
	value1 := creflect.PropertyReflector.GetProperty(myObjectA, "Param2")
	creflect.PropertyReflector.SetProperty(myObjectA, "param2", 14785)
	value2 := creflect.PropertyReflector.GetProperty(myObjectA, "Param2")
	fmt.Println("The value of param2 is: ", value1)
	fmt.Println("The new value of param2 is: ", value2)

}

```

#### RecursiveObjectReader

```
// Recursive Object Reader

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	ObjectAa

	Param1 string
	Param2 int
}

type ObjectAa struct {
	Param5 string
}

func (c *ObjectA) MethodA() int {
	return 123
}

type ObjectB struct {
	*ObjectA
	Param4 string
}

func NewObjectB() *ObjectB {
	return &ObjectB{
		ObjectA: NewObjectA(),
		Param4:  "inside 2",
	}
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
		ObjectAa: ObjectAa{
			Param5: "hello aa",
		},
	}
}

func main() {
	myObjectA := NewObjectA()
	myObjectB := NewObjectB()

	value1 := creflect.RecursiveObjectReader.GetPropertyNames(myObjectA)
	fmt.Println("The property names of myObjectA are: ", value1)

	value2 := creflect.RecursiveObjectReader.HasProperty(myObjectB, "param5")
	fmt.Println("myObjectB contains param5: ", value2)

	value3 := creflect.RecursiveObjectReader.GetProperties(myObjectB)
	fmt.Println("The properties of myObjectB are: ", value3)

	value4 := creflect.RecursiveObjectReader.GetProperty(myObjectB, "Param4")
	fmt.Println("The value of param4 is: ", value4)
}

```

#### RecursiveObjectWriter[ ](http://docs.pipservices.org/v3/tutorials/beginner_tutorials/component/reflection/#frecursiveobjectwriter)

```
// Recursive Object Reader

// RecursiveObjectWriter

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	ObjectAa

	Param1 string
	Param2 int
}

type ObjectAa struct {
	Param5 string
}

func (c *ObjectA) MethodA() int {
	return 123
}

type ObjectB struct {
	*ObjectA
	Param4 string
}

func NewObjectB() *ObjectB {
	return &ObjectB{
		ObjectA: NewObjectA(),
		Param4:  "inside 2",
	}
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
		ObjectAa: ObjectAa{
			Param5: "hello aa",
		},
	}
}

func main() {
	myObjectB := NewObjectB()
	myObjectC := NewObjectB()

	// set_property
	creflect.RecursiveObjectWriter.SetProperty(myObjectB, "Param2", "new value")
	value1 := creflect.RecursiveObjectReader.GetProperty(myObjectB, "Param2")
	fmt.Println("The new values for the myObjectB object are:", value1)

	// set_properties
	myMap := map[string]interface{}{"Param1": 789456, "Param2": "ABCaccc"}
	creflect.RecursiveObjectWriter.SetProperties(myObjectB, myMap)
	value2 := creflect.RecursiveObjectReader.GetProperties(myObjectB)
	fmt.Println("The new values for the myObjectB object are:", value2)

	// copy_proerties
	value3 := creflect.RecursiveObjectReader.GetProperties(myObjectC)
	fmt.Println("The properties of myObjectC and their values are:", value3)
	creflect.RecursiveObjectWriter.CopyProperties(myObjectC, myObjectB)
	value4 := creflect.RecursiveObjectReader.GetProperties(myObjectC)
	fmt.Println("The new properties of myObjectC and their values are:", value4)
}

```

#### TypeDescriptor

```
/// TypeDescriptor

package main

import (
	"fmt"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	ObjectAa

	Param1 string
	Param2 int
}

type ObjectAa struct {
	Param5 string
}

func (c *ObjectA) MethodA() int {
	return 123
}

type ObjectB struct {
	*ObjectA
	Param4 string
}

func NewObjectB() *ObjectB {
	return &ObjectB{
		ObjectA: NewObjectA(),
		Param4:  "inside 2",
	}
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
		ObjectAa: ObjectAa{
			Param5: "hello aa",
		},
	}
}

func main() {
	// Create type descriptors
	type1 := creflect.NewTypeDescriptor("ObjectA", "library1")
	type2 := creflect.NewTypeDescriptor("ObjectB", "library1")

	// equals
	result1 := type1.Equals(type2)
	fmt.Println("type1 equals type2:", result1)

	// get_library
	library1 := type1.Package()
	fmt.Println("The library of type1:", library1)

	// get_name
	name1 := type1.Name()
	fmt.Println("The name of type1 is:", name1)

	// from_string
	typeDescriptor, _ := creflect.ParseTypeDescriptorFromString("ObjectA,library1")
	fmt.Println("Type descriptor:", typeDescriptor)
}

```

#### TypeMatcher

```
// TypeMatcher

package main

import (
	"fmt"
	refl "reflect"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	Param1 string
	Param2 int
}

func (c *ObjectA) MethodA() int {
	return 123
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
	}
}

func main() {
	objectA1 := NewObjectA()

	// expected type: Object, actual type: objectA1, actualvalue: objectA1
	type1 := creflect.TypeMatcher.MatchType("Object", refl.TypeOf(objectA1))
	fmt.Println("objectA1 is an object:", type1)

	// expected type: Object, actual type: String
	type2 := creflect.TypeMatcher.MatchTypeByName("Object", refl.TypeOf("string"))
	fmt.Println("String is an object:", type2)

	// expected type: objectA1, expected value: objectA1
	type3 := creflect.TypeMatcher.MatchValue(refl.TypeOf(objectA1), objectA1)
	fmt.Println("objectA1 is of type objectA1:", type3)

	// expected type: Object, actual value: objectA1
	type4 := creflect.TypeMatcher.MatchValueByName("Object", objectA1)
	fmt.Println("ObjectA1 is of type Object:", type4)

	string1 := "Hello World"
	type5 := creflect.TypeMatcher.MatchValueByName("String", string1)
	fmt.Println("string1 is of type String:", type5)
}

```

#### TypeReflector

```
// TypeReflector

package main

import (
	"fmt"
	refl "reflect"

	creflect "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/reflect"
)

// Object
type ObjectA struct {
	Param1 string
	Param2 int
}

func NewObjectA() *ObjectA {
	return &ObjectA{
		Param1: "hello",
		Param2: 123,
	}
}

func main() {
	myObjectA, _ := creflect.TypeReflector.CreateInstanceByType(refl.TypeOf(ObjectA{})) // Constructors with arguments are not supported

	myObjectA.(*ObjectA).Param1 = "hello"
	myObjectA.(*ObjectA).Param2 = 123

	fmt.Println("The values of param1 and param2 are", myObjectA.(*ObjectA).Param1, "and", myObjectA.(*ObjectA).Param2)
}

```
