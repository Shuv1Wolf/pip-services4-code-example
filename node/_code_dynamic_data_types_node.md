```
// Examples
let value1 = new AnyValue(1);
let value2 = new AnyValue(123.456);
let value3 = new AnyValue("123.456");
```

```
import { AnyValue } from "pip-services4-commons-node"
```

```
// Constructor
let value = new AnyValue("123.456");

// Cloning 
let value2 = value.clone();  // Returns an object of type AnyValue with value 123.456

```

```
import { AnyValue } from "pip-services4-commons-node"

let value = new AnyValue("123.456");

let value1 = value.getAsInteger();                  // Returns 123
let value2 = value.getAsLong();                     // Returns 123
let value3 = value.getAsFloat();                    // Returns 123.456   

let valueB = new AnyValue("ABC");
let value4 = valueB.getAsIntegerWithDefault(25);    // Returns 25

let value5 = value.getAsString();                   // Returns '123.456'
let value6 = value.getAsStringWithDefault("ABC");   // Returns '123.456'

valueB = new AnyValue("1");
let value7 = valueB.getAsBoolean();                 // Returns True

let valueC = new AnyValue("abc");
let value8 = valueC.getAsBooleanWithDefault(false); // Returns False

let type1 = value.getTypeCode();                    // Returns 1 (TypeCode.String)
```

```
let value = new AnyValue("ABC");
value.setAsObject("CEF");       // Sets value to "CEF"

```

```
let value = new AnyValue(123.456);
let value1 = value.toString();    // Returns '123.456'

```

```
import { AnyValue, TypeCode } from "pip-services4-commons-node"

let value = new AnyValue("123.456");
let value2 = value.clone();

let result1 = value.equals(value2);             // Returns True

let result2 = value.equalsAsType(TypeCode.Object, value2);   // Returns True
```

```
let value = new AnyValueArray([ 1, "ABC", 123.45, new Date() ]);
```

```
import { AnyValueArray } from "pip-services4-commons-node"
```

```
// Constructor
let value1 = new AnyValueArray([ 1, 2, 3 ]);

// String
let myString = "1.2.3";
let value2 = AnyValueArray.fromString(myString, '.');

// List
let myList = [1, 2, 3];
let value = AnyValueArray.fromValue(myList);

// Cloning
value2 = value.clone();  // Returns value2 as AnyValueArry with values 1,2,3
```

```
// Find out if it contains a value
let value = new AnyValueArray([ 1, "123.456", "2018-01-01" ]);

let res = value.contains(1); // Returns True

let result = value.containsAsType(TypeCode.Integer, 1);   // Returns True

```

```
import { AnyValueArray, AnyValueMap, TypeCode } from "pip-services4-commons-node"

let value = new AnyValueArray([ 1, "123.456", "2018-01-01" ]);

// Get a value for a specified index
let value1 = value.get(0);  // Returns 1, type int

let value2 = value.getAsBoolean(0); // Returns True
let value3 = value.getAsBooleanWithDefault(1, false);   // Returns False
let value4 = value.getAsNullableBoolean(2); // Returns null

let value5 = value.getAsInteger(1); // Returns 123
let value6 = value.getAsIntegerWithDefault(3, 0);   // Returns 0
let value7 = value.getAsNullableInteger(3); // Returns null

let value8 = value.getAsLong(1);    // Returns 123
let value9 = value.getAsLongWithDefault(3, 0);  // Returns 0
let value10 = value.getAsNullableLong(3);   // Returns null

let value11 = value.getAsFloat(1);  // Returns 123.456
let value12 = value.getAsFloatWithDefault(3, 0.0); // Returns 0.0
let value13 = value.getAsNullableFloat(3);  // Returns null

let value14 = value.getAsDouble(1); // Returns 123.456
let value15 = value.getAsDoubleWithDefault(3, 0.0); // Returns 0.0
let value16 = value.getAsNullableDouble(3); // Returns null

let value17 = value.getAsDateTime(2);   // Returns 2018-01-01 00:00:00+00:00

let value18 = value.getAsDateTimeWithDefault(1, new Date());  // Returns (e.g) 2021-11-04

let value19 = value.getAsString(2); // Returns '2018-01-01'
let value20 = value.getAsNullableString(2); // Returns '2018-01-01'
let value21 = value.getAsStringWithDefault(2, "0000-00-00");    // Returns '2018-01-01'

let value22 = value.getAsArray(1);  // Returns ['123.456']
let value23 = value.getAsArrayWithDefault(0, new AnyValueArray([0]));   // Returns [1]
let value24 = value.getAsNullableArray(2);  // Returns ['2018-01-01']

let valueA = new AnyValueArray([1, { "number": "123.456" }, "2018-01-01" ]);
let value25 = valueA.getAsMap(1);   // Returns {'number': '123.456'}
value25 = valueA.getAsMapWithDefault(3, new AnyValueMap({ "key1": 1 })); // Returns {'key1': 1}
let value27 = valueA.getAsNullableMap(3);   // Returns null


let value28 = value.getAsType(TypeCode.DateTime, 2); // Returns 2018-01-01
let value29 = value.getAsTypeWithDefault(TypeCode.DateTime, 1, new Date());    // Returns today date
let value30 = value.getAsNullableType(TypeCode.DateTime, 1); // Returns null
```

```
// Replace a value
let value = AnyValueArray([1, 2, 3]);
value.put(0,"ABC");    // Returns ABC,2,3

// Append a value
let myList = [1,3,4];
value.append(my_list);  // Returns ABC,2,3,1,3,4

```

```
let value = new AnyValueArray([ 1, "123.456", "2018-01-01" ]);
value.remove(0);    // Returns 123.456,2018-01-01

```

```
let value = new AnyValueMap({
    "key1": 1,
    "key2": 123.456,
    "key3": "ABC",
    "key4": new Date()
});
```

```
import { AnyValueMap } from "pip-services4-commons-node"
```

```
// Constructor
let value = new AnyValueMap({
    "key1": 1,
    "key2": "123.456",
    "key3": "2018-01-01"
}); // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Cloning
let value2 = value.clone();   // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Maps
let myMap1 = new AnyValueMap({
    "key1": 1 ,
    "key2": "123.456" ,
    "key3": "2018-01-01" 
});

let myMap2 = {
    "key4": 2,
    "key5": 3,
    "key6": 4,
};

value = AnyValueMap.fromMaps(myMap1, myMap2);    // Returns {"key1": 1,"key2": "123.456","key3": "2018-01-01","key4": 2,"key5": 3,"key6": 4} 

// Tuples
value = AnyValueMap.fromTuples("key1", 1, "key2", "123.456", "key3", "2018-01-01");     // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Array of tuples
let myTuple = [ "key1", 1, "key2", "123.456", "key3", "2018-01-01" ];
value = AnyValueMap.fromTuplesArray(myTuple);  // Returns {"key1": 1, "key2": "123.456", "key3": "2018-01-01"}

// Values
let myValue = [1,2];
value = AnyValueMap.fromValue(myValue);       // Returns {"0": 1, "1": 2}

```

```

let value = new AnyValueMap({
    "key1": 1,
    "key2": "123.456",
    "key3": "2018-01-01",
    "key4": "word"
});

let value1 = value.getAsBoolean("key1");   // Returns: true
let value2 = value.getAsInteger("key2");   // Returns: 123
let value3 = value.getAsIntegerWithDefault("key4", 0);   // Returns 0
let value4 = value.getAsFloat("key2");     // Returns: 123.456
let value5 = value.getAsDateTime("key3");  // Returns new 2018-0-1
let valueA = new AnyValueMap({ 
    "key1": 1,
    "key2":  { "key": "123.456" },
    "key3": "2018-01-01" 
}); // redact

let value6 = valueA.getAsMap("key2");      // Returns {'key': '123.456'}
let value7 = value.getAsNullableDateTime("key2");     // Returns null
let value8 = value.getAsNullableDateTime("key3");     // Returns 2018-0-1
let value9 = value.getAsString("key1");    // Returns '1'
let value10 = value.getAsObject();                     // Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}

let value11 = value.getAsType(TypeCode.String, "key1");     // Returns '1'
let value12 = value.getAsValue("key1");

```

```
let value = new AnyValueMap({
    "key1": 1 ,
    "key2": "123.456",
    "key3": "2018-01-01" 
});

value.setAsObject("key1", 2);          // Returns {"key1": 2, "key2": "123.456", "key3": "2018-01-01"}


```

```
var value = new AnyValueMap({
    "key1": 1,
    "key2": "123.456",
    "key3": "2018-01-01" 
});
value.remove("key1");        // Returns value = {'key2': '123.456', 'key3': '2018-01-01'}


```
