```
// Array converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.ArrayConverter.ToArray([]int{1, 2}) // Returns [1, 2]
	value2 := convert.ArrayConverter.ToArray(1)           // Returns [1]

	defaultValue := make([]interface{}, 0)
	defaultValue = append(defaultValue, 1)

	value3 := convert.ArrayConverter.ToArrayWithDefault(nil, defaultValue) // Returns [1]
	value4 := convert.ArrayConverter.ListToArray("1,2,3")                  // Returns ['1', '2', '3']
	value5, err := convert.ArrayConverter.ToNullableArray("1,2")                // Returns ['1,2']

}
```

```
// Boolean converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.BooleanConverter.ToBoolean("yes")                          // Returns True
	value2 := convert.BooleanConverter.ToBooleanWithDefault(true, false)         // Returns True
	value3 := convert.BooleanConverter.ToBooleanWithDefault(123, false)          // Returns False
	value4, err := convert.BooleanConverter.ToNullableBoolean(true)                  // Returns True
	value5, err := convert.BooleanConverter.ToNullableBoolean("yes")                 // Returns True
	value6, err := convert.BooleanConverter.ToNullableBoolean(123)                    // Returns nil
	value7, err := convert.BooleanConverter.ToNullableBoolean(make([]interface{}, 0)) // Returns nil
}
```

```
// Date time converter

// Returns 2021-11-09 17:24:50.75
value0 := convert.DateTimeConverter.ToDateTime("2021-11-09T17:24:50.750Z")

// Returns nil
value1, err := convert.DateTimeConverter.ToNullableDateTime("ABC")

// Returns 2021-11-09 17:24:50.75
value2, err := convert.DateTimeConverter.ToNullableDateTime("2021-11-09T17:24:50.750Z")

// Returns 2361-03-21 16:22:35
value3, err := convert.DateTimeConverter.ToNullableDateTime(12345657755000)

// Returns current datetime
value4 := convert.DateTimeConverter.ToDateTimeWithDefault("ABC", time.Now())

// Returns 2021-11-09 17:24:50.75
value5 := convert.DateTimeConverter.ToDateTimeWithDefault("2021-11-09T17:24:50.750Z", time.Now())
```

```
// Double converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.DoubleConverter.ToDouble("123.456")               // Returns 123.456
	value2 := convert.DoubleConverter.ToDouble("ABC")                   // Returns 0
	value3 := convert.DoubleConverter.ToDoubleWithDefault("123.456", 0) // Returns 123.456
	value4 := convert.DoubleConverter.ToDoubleWithDefault("ABC", 0)     // Returns 0
	value5, err := convert.DoubleConverter.ToNullableDouble("123.456")  // Returns 123.456
	value6, err := convert.DoubleConverter.ToNullableDouble("ABC")      // Returns nil
	value7, err := convert.DoubleConverter.ToNullableDouble(true)       // Returns 1
}
```

```
// Float converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.FloatConverter.ToFloat("123.456")               // Returns 123.456
	value2 := convert.FloatConverter.ToFloat("ABC")                   // Returns 0
	value3 := convert.FloatConverter.ToFloatWithDefault("123.456", 0) // Returns 123.456
	value4 := convert.FloatConverter.ToFloatWithDefault("ABC", 0)     // Returns 0
	value5, err := convert.FloatConverter.ToNullableFloat("123.456")  // Returns 123.456
	value6, err := convert.FloatConverter.ToNullableFloat("ABC")      // Returns nil
	value7, err := convert.FloatConverter.ToNullableFloat(true)       // Returns 1
}
```

```
// IntegerConverter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.IntegerConverter.ToInteger("123.456")                 // Returns 123
	value2 := convert.IntegerConverter.ToInteger("ABC")                     // Returns 0
	value3, err := convert.IntegerConverter.ToNullableInteger("123.456")    // Returns 123
	value4, err := convert.IntegerConverter.ToNullableInteger("ABC")        // Returns nil
	value5, err := convert.IntegerConverter.ToNullableInteger(true)         // Returns 1
	value6 := convert.IntegerConverter.ToIntegerWithDefault("ABC", 123)     // Returns 123
	value7 := convert.IntegerConverter.ToIntegerWithDefault("123.456", 123) // Returns 123
}
```

```
// Long converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.LongConverter.ToLong("123.456")               // Returns 123
	value2 := convert.LongConverter.ToLong("ABC")                   // Returns 0
	value3, err := convert.LongConverter.ToNullableLong("123.456")  // Returns 123
	value4, err := convert.LongConverter.ToNullableLong("ABC")      // Returns nil
	value5, err := convert.LongConverter.ToNullableLong(true)       // Returns 1
	value6 := convert.LongConverter.ToLongWithDefault("123.456", 0) // Returns 123
	value7 := convert.LongConverter.ToLongWithDefault("ABC", 0)     // Returns 0
}
```

```
// String converter

import (
	"time"

	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.StringConverter.ToString(123.456)                                      // Returns '123.456'
	value2 := convert.StringConverter.ToString(true)                                         // Returns 'True'
	value3 := convert.StringConverter.ToString(time.Date(2018, 10, 1, 0, 0, 0, 0, time.UTC)) // Returns '2018-10-01T00:00:00Z'
	value4 := convert.StringConverter.ToString([]string{"a", "b", "c"})                      // Returns 'a,b,c'
	value5 := convert.StringConverter.ToString(nil)                                          // Returns ""
	value6, err := convert.StringConverter.ToNullableString(nil)                             // Returns nil
	value7 := convert.StringConverter.ToStringWithDefault(nil, "my default")                 // Returns 'my default'
}

```

```
// JSON converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1, _ := convert.JsonConverter.ToJson(map[string]int{"key": 123})                                // Returns '{"key": 123}'
	value2, _ := convert.JsonConverter.FromJson("{\"key\":\"123\"}")                                     // Returns {"key": 123}
	value3 := convert.JsonConverter.ToMap(value1)                                                        // Returns {"key": 123}
	value4 := convert.JsonConverter.ToMapWithDefault(value1, map[string]interface{}{"my key": "my val"}) // Returns {"key": 123}
	value5 := convert.JsonConverter.ToMapWithDefault("", map[string]interface{}{"my key": "my val"})     // Returns {}
	value6, _ := convert.JsonConverter.ToNullableMap(value1)                                             // Returns {'key': 123}
	value7, _ := convert.JsonConverter.ToNullableMap("{}")                                               // Returns {}
}
```

```
// Map converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1, _ := convert.MapConverter.ToNullableMap("ABC")                                        // Returns nil
	value2, _ := convert.MapConverter.ToNullableMap(map[string]interface{}{"key": 123})           // Returns { key: 123 }
	value3, _ := convert.MapConverter.ToNullableMap([]int{1, 2, 3})                               // Returns { "0": 1, "1": 2, "2": 3 }
	value4 := convert.MapConverter.ToMap("ABC")                                                   // Returns {}
	value5 := convert.MapConverter.ToMapWithDefault("ABC", value2)                                // Returns {'key': 123}
	value6 := convert.MapConverter.ToMapWithDefault(map[string]interface{}{"key": 12345}, value2) // Returns {'key': 12345}
}
```

```
// Recursive map ???????
```

```
// Type converter

import (
	convert "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
)

func main() {
	value1 := convert.TypeConverter.ToType(convert.String, 123)                  // Returns '123'
	value2, _ := convert.TypeConverter.ToNullableType(convert.String, 123)       // Returns '123'
	value3 := convert.TypeConverter.ToTypeWithDefault(convert.Integer, "ABC", 1) // Returns 1
	value4 := convert.TypeConverter.ToTypeCode("Hello world")                    // Returns TypeCode.String
	value5 := convert.TypeConverter.ToString(convert.String)                     // Returns 'string'
}
```
