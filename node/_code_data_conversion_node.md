```
import { ArrayConverter } from "pip-services4-commons-node"

let value1 = ArrayConverter.toArray([1, 2]);                // Returns [1, 2]
let value2 = ArrayConverter.toArray(1);                     // Returns [1]
let value3 = ArrayConverter.toArrayWithDefault(null, [1]);  // Returns [1]
let value4 = ArrayConverter.listToArray("1,2,3");           // Returns ['1', '2', '3']
let value5 = ArrayConverter.toNullableArray("1,2");         // Returns ['1,2']
```

```
import { BooleanConverter } from "pip-services4-commons-node"

let value1 = BooleanConverter.toBoolean("yes");                     // Returns True
let value2 = BooleanConverter.toBooleanWithDefault(true, false);    // Returns True
let value3 = BooleanConverter.toBooleanWithDefault(123, false);     // Returns True
let value4 = BooleanConverter.toNullableBoolean(true);              // Returns True
let value5 = BooleanConverter.toNullableBoolean("yes");             // Returns True
let value6 = BooleanConverter.toNullableBoolean(123);               // Returns True
let value7 = BooleanConverter.toNullableBoolean({});                // Returns null
```

```
import { DateTimeConverter } from "pip-services4-commons-node"

let value0 = DateTimeConverter.toDateTime("2021-11-09T17:24:50.750Z");          // Returns 2021-11-09 17:24:50.75
let value1 = DateTimeConverter.toNullableDateTime("ABC");                       // Returns null
let value2 = DateTimeConverter.toNullableDateTime("2021-11-09T17:24:50.750Z");  // Returns 2021-11-09 17:24:50.75
let value3 = DateTimeConverter.toNullableDateTime(12345657755000);              // Returns 2361-03-21 16:22:35
let value4 = DateTimeConverter.toDateTimeWithDefault("ABC", new Date());        // Returns current datetime
let value5 = DateTimeConverter.toDateTimeWithDefault("2021-11-09T17:24:50.750Z", new Date()); // Returns 2021-11-09 17:24:50.75


```

```
import { DoubleConverter } from "pip-services4-commons-node"

let value1 = DoubleConverter.toDouble("123.456");               // Returns 123.456
let value2 = DoubleConverter.toDouble("ABC");                   // Returns 0
let value3 = DoubleConverter.toDoubleWithDefault("123.456", 0); // Returns 123.456
let value4 = DoubleConverter.toDoubleWithDefault("ABC", 0);     // Returns 0
let value5 = DoubleConverter.toNullableDouble("123.456");       // Returns 123.456
let value6 = DoubleConverter.toNullableDouble("ABC");           // Returns null
let value7 = DoubleConverter.toNullableDouble(true);            // Returns 1

```

```
import { FloatConverter } from "pip-services4-commons-node"

let value1 = FloatConverter.toFloat("123.456");                 // Returns 123.456
let value2 = FloatConverter.toFloat("ABC");                     // Returns 0
let value3 = FloatConverter.toFloatWithDefault("123.456", 0);   // Returns 123.456
let value4 = FloatConverter.toFloatWithDefault("ABC", 0);       // Returns 0
let value5 = FloatConverter.toNullableFloat("123.456");         // Returns 123.456
let value6 = FloatConverter.toNullableFloat("ABC");             // Returns null
let value7 = FloatConverter.toNullableFloat(true);              // Returns 1

```

```
import { IntegerConverter } from "pip-services4-commons-node"

let value1 = IntegerConverter.toInteger("123.456");                 // Returns 123
let value2 = IntegerConverter.toInteger("ABC");                     // Returns 0
let value3 = IntegerConverter.toNullableInteger("123.456");         // Returns 123
let value4 = IntegerConverter.toNullableInteger("ABC");             // Returns null
let value5 = IntegerConverter.toNullableInteger(true);              // Returns 1
let value6 = IntegerConverter.toIntegerWithDefault("ABC", 123);     // Returns 123 
let value7 = IntegerConverter.toIntegerWithDefault("123.456", 123); // Returns 123 

```

```
import { LongConverter } from "pip-services4-commons-node"

let value1 = LongConverter.toLong("123.456");               // Returns 123
let value2 = LongConverter.toLong("ABC");                   // Returns 0
let value3 = LongConverter.toNullableLong("123.456");       // Returns 123
let value4 = LongConverter.toNullableLong("ABC");           // Returns null
let value5 = LongConverter.toNullableLong(true);            // Returns 1
let value6 = LongConverter.toLongWithDefault("123.456", 0); // Returns 123
let value7 = LongConverter.toLongWithDefault("ABC", 0);     // Returns 0

```

```
import { StringConverter } from "pip-services4-commons-node"

let value1 = StringConverter.toString(123.456);                 // Returns '123.456'
let value2 = StringConverter.toString(true);                    // Returns 'True'
let value3 = StringConverter.toString(new Date(2018, 10, 1));   // Returns '2018-10-01T00:00:00Z'
let value4 = StringConverter.toString([ "a", "b", "c" ]);       // Returns 'a,b,c'
let value5 = StringConverter.toString(null);                    // Returns ""
let value6 = StringConverter.toNullableString(null);            // Returns null
let value7 = StringConverter.toStringWithDefault(null, "my default");   // Returns 'my default' 

```

```
import { JsonConverter, TypeCode } from "pip-services4-commons-node"

let value1 = JsonConverter.toJson({ "key": 123 });                           // Returns '{"key": 123}'
let value2 = JsonConverter.fromJson(TypeCode.Map, "{\"key\":\"123\"}");      // Returns {'key': '123'}
let value3 = JsonConverter.toMap(value1);                                    // Returns {'key': 123}
let value4 = JsonConverter.toMapWithDefault(value1, { "my key": "my val" }); // Returns {'key': 123}
let value5 = JsonConverter.toMapWithDefault("", { "my key": "my val" });     // Returns { "my key": "my val" }
let value6 = JsonConverter.toNullableMap(value1);                            // Returns {'key': 123}
let value7 = JsonConverter.toNullableMap("{}");                              // Returns {}
  
```

```
import { MapConverter } from "pip-services4-commons-node"

let value1 = MapConverter.toNullableMap("ABC");                       // Returns null
let value2 = MapConverter.toNullableMap({ "key": 123 });              // Returns { key: 123 }
let value3 = MapConverter.toNullableMap([1, 2, 3]);                   // Returns { "0": 1, "1": 2, "2": 3 }
let value4 = MapConverter.toMap("ABC");                               // Returns {}
let value5 = MapConverter.toMapWithDefault("ABC", value2);            // Returns {'key': 123}
let value6 = MapConverter.toMapWithDefault({ "key": 12345 }, value2); // Returns {'key': 12345}

```

```
import { RecursiveMapConverter } from "pip-services4-commons-node"

let value1 = RecursiveMapConverter.toMap({ "key": 123});                              // Returns {'key': 123}
let value2 = RecursiveMapConverter.toMapWithDefault(null, { "my key": "my val" });    // Returns { "my key": "my val" }
let value3 = RecursiveMapConverter.toNullableMap({ "key": 123 });                     // Returns {'key': 123}
let value4 = RecursiveMapConverter.toNullableMap([1,[2,3]]); // Returns {0: 1, 1: {0: 2, 1: 3}}

```

```
import { TypeConverter, TypeCode } from "pip-services4-commons-node"

let value1 = TypeConverter.toType(TypeCode.String, 123);                  // Returns '123'
let value2 = TypeConverter.toNullableType(TypeCode.String, 123);          // Returns '123'
let value3 = TypeConverter.toTypeWithDefault(TypeCode.Integer, "ABC", 1); // 1
let value4 = TypeConverter.toTypeCode("Hello world");                     // Returns TypeCode.String
let value5 = TypeConverter.toString(TypeCode.String);                     // Returns 'string'

```
