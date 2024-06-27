```python
# Array converter

from pip_services4_commons.convert import ArrayConverter

value1 = ArrayConverter.to_array([1, 2])       # Returns [1, 2]
value2 = ArrayConverter.to_array(1)            # Returns [1]
value3 = ArrayConverter.to_array_with_default(None,[1])      # Returns [1]
value4 = ArrayConverter.list_to_array("1,2,3")               # Returns ['1', '2', '3']
value5 = ArrayConverter.to_nullable_array('1,2')             # Returns ['1,2']
```

```python
# Boolean converter

from pip_services4_commons.convert import BooleanConverter

value1 = BooleanConverter.to_boolean("yes")            # Returns True
value2 = BooleanConverter.to_boolean_with_default(True, False)     # Returns True   
value3 = BooleanConverter.to_boolean_with_default(123, False)      # Returns False
value4 = BooleanConverter.to_nullable_boolean(True)    # Returns True
value5 = BooleanConverter.to_nullable_boolean("yes")   # Returns True
value6 = BooleanConverter.to_nullable_boolean(123)     # Returns None
value7 = BooleanConverter.to_nullable_boolean({})      # Returns None


```

```python
# Date time converter

from pip_services4_commons.convert import DateTimeConverter
from datetime import date

value0 = DateTimeConverter.to_datetime("2021-11-09T17:24:50.750Z")              # Returns 2021-11-09 17:24:50.75
value1 = DateTimeConverter.to_nullable_datetime("ABC")                          # Returns None
value2 = DateTimeConverter.to_nullable_datetime("2021-11-09T17:24:50.750Z")     # Returns 2021-11-09 17:24:50.75
value3 = DateTimeConverter.to_nullable_datetime(12345657755000)                 # Returns 2361-03-21 16:22:35
value4 = DateTimeConverter.to_datetime_with_default("ABC", date.today())        # Returns current datetime
value4 = DateTimeConverter.to_datetime_with_default("2021-11-09T17:24:50.750Z", date.today())   # Returns 2021-11-09 17:24:50.75

```

```python
# Double converter

from pip_services4_commons.convert import DoubleConverter

value1 = DoubleConverter.to_double("123.456")                   # Returns 123.456
value2 = DoubleConverter.to_double("ABC")                       # Returns 0
value3 = DoubleConverter.to_double_with_default("123.456", 0)   # Returns 123.456
value4 = DoubleConverter.to_double_with_default("ABC", 0)       # Returns 0
value5 = DoubleConverter.to_nullable_double("123.456")       # Returns 123.456
value6 = DoubleConverter.to_nullable_double("ABC")           # Returns None
value7 = DoubleConverter.to_nullable_double(True)            # Returns 1

```

```python
# Float converter
from pip_services4_commons.convert import FloatConverter

value1 = FloatConverter.to_float("123.456")                 # Returns 123.456
value2 = FloatConverter.to_float("ABC")                     # Returns 0
value3 = FloatConverter.to_float_with_default("123.456", 0) # Returns 123.456
value4 = FloatConverter.to_float_with_default("ABC", 0)     # Returns 0
value5 = FloatConverter.to_nullable_float("123.456")        # Returns 123.456
value6 = FloatConverter.to_nullable_float("ABC")            # Returns None
value7 = FloatConverter.to_nullable_float(True)             # Returns 1


```

```python
# IntegerConverter
from pip_services4_commons.convert import IntegerConverter

value1 = IntegerConverter.to_integer("123.456")                     # Returns 123
value2 = IntegerConverter.to_integer("ABC")                         # Returns 0
value3 = IntegerConverter.to_nullable_integer("123.456")            # Returns 123
value4 = IntegerConverter.to_nullable_integer("ABC")                # Returns None
value5 = IntegerConverter.to_nullable_integer(True)                 # Returns 1
value6 = IntegerConverter.to_integer_with_default("ABC", 123)       # Returns 123  
value7 = IntegerConverter.to_integer_with_default("123.456", 123)   # Returns 123  


```

```python
# Long converter

from pip_services4_commons.convert import LongConverter

value1 = LongConverter.to_long("123.456")                   # Returns 123
value2 = LongConverter.to_long("ABC")                       # Returns 0
value3 = LongConverter.to_nullable_long("123.456")          # Returns 123
value4 = LongConverter.to_nullable_long("ABC")              # Returns None
value5 = LongConverter.to_nullable_long(True)               # Returns 1
value6 = LongConverter.to_long_with_default("123.456", 0)   # Returns 123
value7 = LongConverter.to_long_with_default("ABC", 0)       # Returns 0

```

```python
# String converter

from pip_services4_commons.convert import StringConverter
import datetime

value1 = StringConverter.to_string(123.456)     # Returns '123.456'
value2 = StringConverter.to_string(True)        # Returns 'True'
value3 = StringConverter.to_string(datetime.datetime(2018,10,1))       # Returns '2018-10-01T00:00:00Z'
value4 = StringConverter.to_string(["a","b","c"])     # Returns 'a,b,c'
value5 = StringConverter.to_string(None)              # Returns ""
value6 = StringConverter.to_nullable_string(None)     # Returns None
value7 = StringConverter.to_string_with_default(None,"my default")     # Returns 'my default' 

```

```python
# JSON converter
from pip_services4_commons.convert import JsonConverter, TypeCode

value1 = JsonConverter.to_json({'key': 123})                                # Returns '{"key": 123}'
value2 = JsonConverter.from_json(TypeCode.Map, '{"key":"123"}')          # Returns {'key': '123'}
value3 = JsonConverter.to_map(value1)                                       # Returns {'key': 123}
value4 = JsonConverter.to_map_with_default(value1, {"my key": "my val"})    # Returns {'key': 123}
value5 = JsonConverter.to_map_with_default("", {"my key": "my val"})        # Returns {"my key": "my val"}
value6 = JsonConverter.to_nullable_map(value1)                              # Returns {'key': 123}
value7 = JsonConverter.to_nullable_map("")                                  # Returns None

```

```python
# Map converter

from pip_services4_commons.convert import MapConverter

value1 = MapConverter.to_nullable_map("ABC")                        # Returns None
value2 = MapConverter.to_nullable_map({ 'key': 123 })               # Returns { key: 123 }
value3 = MapConverter.to_nullable_map([1,2,3])                      # Returns { "0": 1, "1": 2, "2": 3 }
value4 = MapConverter.to_map("ABC")                                 # Returns {}
value5 = MapConverter.to_map_with_default("ABC", value2)            # Returns {'key': 123}
value6 = MapConverter.to_map_with_default({ 'key': 12345 }, value2) # Returns {'key': 12345}


```

```python
# Recursive map
from pip_services4_commons.convert import RecursiveMapConverter

value1 = RecursiveMapConverter.to_map({ 'key': 123 })              # Returns {'key': 123}
value2 = RecursiveMapConverter.to_map_with_default(None, { "my key": "my val" })   # Returns { "my key": "my val" }
value3 = RecursiveMapConverter.to_nullable_map({ 'key': 123 })     # Returns {'key': 123}
value4 = RecursiveMapConverter.to_nullable_map([1,[2,3]])          # Returns {0: 1, 1: {0: 2, 1: 3}}

```

```python
# Type converter

from pip_services4_commons.convert import TypeCode

value1 = TypeConverter.to_type(TypeCode.String, 123)             # Returns '123'
value2 = TypeConverter.to_nullable_type(TypeCode.String, 123)    # Returns '123'
value3 = TypeConverter.to_type_with_default(TypeCode.Integer, 'ABC', 1)    # 1
value4 = TypeConverter.to_type_code("Hello world")               # Returns TypeCode.String
value5 = TypeConverter.to_string(TypeCode.String)                # Returns 'string'

```
