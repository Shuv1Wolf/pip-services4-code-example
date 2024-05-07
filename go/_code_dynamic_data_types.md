```

// Examples
value1 := data.NewAnyValue(1)
value2 := data.NewAnyValue(123.456)
value3 := data.NewAnyValue("123.456")
```

```
import (
	data "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)
```

```
// Constructor
value := data.NewAnyValue("123.456")

// Cloning
value2 := value.Clone() // Returns an object of type AnyValue with value 123.456
```

```
import (
	"fmt"

	data "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)

func main() {

	value := data.NewAnyValue("123.456")

	value1 := value.GetAsInteger() // Returns 123
	fmt.Println(value1)
	value2 := value.GetAsLong() // Returns 123
	fmt.Println(value2)
	value3 := value.GetAsFloat() // Returns 123.456
	fmt.Println(value3)

	valueB := data.NewAnyValue("ABC")
	value4 := valueB.GetAsIntegerWithDefault(25) // Returns 25
	fmt.Println(value4)

	value5 := value.GetAsString() // Returns '123.456'
	fmt.Println(value5)
	value6 := value.GetAsStringWithDefault("ABC") // Returns '123.456'
	fmt.Println(value6)

	valueB = data.NewAnyValue("1")
	value7 := valueB.GetAsBoolean() // Returns True
	fmt.Println(value7)

	valueC := data.NewAnyValue("abc")
	value8 := valueC.GetAsBooleanWithDefault(false) // Returns False
	fmt.Println(value8)

	type1 := value.TypeCode() // Returns TypeCode.String 1
	fmt.Println(type1)
}
```

```
value := data.NewAnyValue("ABC")
value.SetAsObject("CEF") // Sets value to "CEF"
```

```

value := data.NewAnyValue(123)
value.String() // Returns '123.456'
```

```
import (
	"fmt"

	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	data "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)

func main() {
	value := data.NewAnyValue("123.456")
	value2 := value.Clone()

	result1 := value.Equals(value2) // Returns True
	fmt.Println(result1)

	result2 := value.EqualsAsType(convert.Object, value2) // Returns True
	fmt.Println(result2)
}
```

```
value := data.NewAnyValueArray([]interface{}{1, "ABC", 123.45, time.Now()})
```

```
import (
	data "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)
```

```
// Constructor
value1 := data.NewAnyValueArray([]interface{}{1, 2, 3})

// String
myString := "1.2.3"
value2 := data.NewAnyValueArrayFromString(myString, ".", false)

// List
myList := []int{1, 2, 3}
value := data.NewAnyValueArrayFromValues(myList)

// Cloning
value3 := value.Clone() // Returns value2 as AnyValueArry with values 1,2,3
```

```
// Find out if it contains a value
value := data.NewAnyValueArray([]interface{}{1, "123.456", "2018-01-01"})

value.Contains(1) // Returns True
result := value.ContainsAsType(convert.Integer, 1) // Returns True
```

```

value := data.NewAnyValueArray([]interface{}{1, "123.456", "2018-01-01"})

// Get a value for a specified index
value1 := value.Get(0) // Returns 1, type int

value2 := value.GetAsBoolean(0)                   // Returns True
value3 := value.GetAsBooleanWithDefault(1, false) // Returns False
value4 := value.GetAsNullableBoolean(2)           // Returns nil

value5 := value.GetAsInteger(1)               // Returns 123
value6 := value.GetAsIntegerWithDefault(2, 0) // Returns 0
value7 := value.GetAsNullableInteger(2)       // Returns nil

value8 := value.GetAsLong(1)               // Returns 123
value9 := value.GetAsLongWithDefault(2, 0) // Returns 0
value10 := value.GetAsNullableLong(2)      // Returns nil

value11 := value.GetAsFloat(1)                 // Returns 123.456
value12 := value.GetAsFloatWithDefault(2, 0.0) // Returns 0.0
value13 := value.GetAsNullableFloat(2)         // Returns nil

value14 := value.GetAsDouble(1)                 // Returns 123.456
value15 := value.GetAsDoubleWithDefault(2, 0.0) // Returns 0.0
value16 := value.GetAsNullableDouble(2)         // Returns nil

value17 := value.GetAsDateTime(2) // Returns 01-01-01 00:00:00+00:00

value18 := value.GetAsDateTimeWithDefault(1, time.Now()) // Returns (e.g) 2021-11-04 00:00:00+00:00

value19 := value.GetAsString(2)                          // Returns '2018-01-01'
value20 := value.GetAsNullableString(2)                  // Returns '2018-01-01'
value21 := value.GetAsStringWithDefault(2, "0000-00-00") // Returns '2018-01-01'

value22 := value.GetAsArray(1)                                                     // Returns ['123.456']
value23 := value.GetAsArrayWithDefault(0, data.NewAnyValueArray([]interface{}{0})) // Returns [1]
value24 := value.GetAsNullableArray(2)                                             // Returns ['2018-01-01']

valueA := data.NewAnyValueArray([]interface{}{1, map[string]interface{}{"number": "123.456"}, "2018-01-01"})
value25 := valueA.GetAsMap(1)                                                                   // Returns {'number': '123.456'}
value25 = valueA.GetAsMapWithDefault(1, data.NewAnyValueMap(map[string]interface{}{"key1": 1})) // Returns {'key1': 1}
value27 := valueA.GetAsNullableMap(1)                                                           // Returns nil

value28 := value.GetAsType(convert.DateTime, 2)                        // Returns 2018-01-01 00:00:00+00:00
value29 := value.GetAsTypeWithDefault(convert.DateTime, 1, time.Now()) // Returns today date
value30 := value.GetAsNullableType(convert.DateTime, 1)                // Returns nil
```

```

// Replace a value
value := data.NewAnyValueArray([]interface{}{1, 2, 3})
value.Put(0, "ABC") // Returns ABC,2,3

// Append a value
myList := []interface{}{1, 3, 4}
value.Append(myList) // Returns ABC,2,3,1,3,4
```

```
value := data.NewAnyValueArray([]interface{}{1, "123.456", "2018-01-01"})
value.Remove(0) // Returns 123.456,2018-01-01
```

```
value := data.NewAnyValueMap(map[string]interface{}{"key1": 1, "key2": 123.456, "key3": "ABC", "key4": time.Now()})
```

```

import (
	data "github.com/pip-services3-gox/pip-services3-commons-gox/data"
)
```

```
// Constructor
value := data.NewAnyValueMap(map[string]interface{}{"key1": 1, "key2": "123.456", "key3": "2018-01-01"}) // Returns{"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Cloning
value2 := value.Clone() // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Maps
myMap1 := map[string]interface{}{"key1": 1, "key2": "123.456", "key3": "2018-01-01"}
myMap2 := map[string]interface{}{"key4": 2, "key5": 3, "key6": 4}

value = data.NewAnyValueMapFromMaps(myMap1, myMap2) // Returns {"key1": 1,"key2": "123.456","key3": "2018-01-01","key4": 2,"key5": 3,"key6": 4}

// Tuples
value = data.NewAnyValueMapFromTuples("key1", 1, "key2", "123.456", "key3", "2018-01-01") // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Array of tuples
myTuple := []interface{}{"key1", 1, "key2", "123.456", "key3", "2018-01-01"}
value = data.NewAnyValueMapFromTuplesArray(myTuple) // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Values
myValue := []int{1, 2}
value = data.NewAnyValueMapFromValue(myValue) // Returns {"0": 1, "1": 2}
```

```
import (
	"fmt"

        convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	data "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)

func main() {
	value := data.NewAnyValueMap(map[string]any{"key1": 1, "key2": "123.456", "key3": "2018-01-01"})

	value1 := value.GetAsBoolean("key1")                                                                                     // Returns: true
	value2 := value.GetAsInteger("key2")                                                                                     // Returns: 123
	value3 := value.GetAsIntegerWithDefault("key3", 0)                                                                       // Returns 0
	value4 := value.GetAsFloat("key2")                                                                                       // Returns: 123.456
	value5 := value.GetAsDateTime("key3")                                                                                    // Returns new Date(2018,0,1)
	value9 := value.GetAsString("key1")                                                                                      // Returns '1'
	valueA := data.NewAnyValueMap(map[string]any{"key1": 1, "key2": map[string]any{"key": "123.456"}, "key3": "2018-01-01"}) // redact
	value6 := valueA.GetAsMap("key2")                                                                                        // Returns {'key': '123.456'}
	value7, ok := value.GetAsNullableDateTime("key2")                                                                        // Returns None
	value8, ok := value.GetAsNullableDateTime("key3")                                                                        // Returns new Date(2018,0,1)
	value10, ok := value.GetAsObject("")

	value11 := value.GetAsType(convert.String, "key1") // Returns '1'
	value12 := value.GetAsValue("key1")
}
```

```

value := data.NewAnyValueMap(map[string]interface{}{"key1": 1, "key2": "123.456", "key3": "2018-01-01"})
value.SetAsObject("key1", 2) // Returns {"key1": 2, "key2": "123.456", "key3": "2018-01-01"}
```

```
value := data.NewAnyValueMap(map[string]interface{}{"key1": 1, "key2": "123.456", "key3": "2018-01-01"})
value.Remove("key1") /// Returns value = {"key2": "123.456", "key3": "2018-01-01"}

```
