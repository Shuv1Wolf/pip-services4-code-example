```python
# Examples
value1 = AnyValue(1)
value2 = AnyValue(123.456)
value3 = AnyValue("123.456")
```

```python
from pip_services4_commons.data import AnyValue
```

```python
# Constructor
value = AnyValue("123.456")

# Cloning 
value2 = value.clone()  # Returns an object of type AnyValue with value 123.456
```

```python
from pip_services4_commons.data import AnyValue

value = AnyValue("123.456")

value1 = value.get_as_integer()                         # Returns 123
value2 = value.get_as_long()                            # Returns 123
value3 = value.get_as_float()                           # Returns 123.456                     

valueB = AnyValue("ABC")
value4 = valueB.get_as_integer_with_default(25)      # Returns 25

value5 = value.get_as_string()                          # Returns '123.456'
value6 = value.get_as_string_with_default("ABC")        # Returns '123.456'

valueB = AnyValue("1")
value7 = valueB.get_as_boolean()                        # Returns True

valueC = AnyValue("abc")
value8 = valueC.get_as_boolean_with_default(False)      # Returns False

type1 = value.get_type_code()                           # Returns TypeCode.String
```

```python
value = AnyValue("ABC")
value.set_as_object("CEF")    # Sets value to "CEF"
```

```python
value = AnyValue(123.456)
value1 = value.to_string()    # Returns '123.456'
```

```python
value = AnyValue("123.456")
value2 = value.clone()

result1 = value.equals(value2)                           # Returns True

from pip_services4_commons.convert import TypeCode 
result2 = value.equals_as_type(TypeCode.Object, value2)  # Returns True
```

```python
value = AnyValueArray([1, "ABC", 123.45, date.today()])
```

```python
from pip_services4_commons.data import AnyValueArray
```

```python
# Constructor
value1 = AnyValueArray([1, 2, 3])

# String
my_string = "1.2.3"
value2 = AnyValueArray.from_string(my_string, '.')

# List
my_list = [1, 2, 3]
value = AnyValueArray.from_values(my_list)

# Cloning
value2 = value.clone()    # Returns value2 as AnyValueArry with values 1,2,3
```

```python
# Find out if it contains a value
value = AnyValueArray([1, "123.456", "2018-01-01"])

value.contains(1) # Returns True

from pip_services4_commons.convert import TypeCode
result = value.contains_as_type(TypeCode.Integer, 1)   # Returns True
```

```python
from pip_services4_commons.convert import TypeCode 
from pip_services4_commons.data import AnyValueArray, AnyValueMap
from datetime import date

value = AnyValueArray([1, "123.456", "2018-01-01"])

# Get a value for a specified index
value1 = value.get(0)                                   # Returns 1, type int

value2 = value.get_as_boolean(0)                        # Returns True
value3 = value.get_as_boolean_with_default(1, False)     # Returns False
value4 = value.get_as_nullable_boolean(2)               # Returns None

value5 = value.get_as_integer(1)                        # Returns 123
value6 = value.get_as_integer_with_default(1, 0)        # Returns 0
value7 = value.get_as_nullable_integer(1)               # Returns None

value8 = value.get_as_long(1)                           # Returns 123
value9 = value.get_as_long_with_default(2, 0)           # Returns 0
value10 = value.get_as_nullable_long(2)                 # Returns None

value11 = value.get_as_float(1)                         # Returns 123.456
value12 = value.get_as_float_with_default(2, 0.0)       # Returns 0.0
value13 = value.get_as_nullable_float(2)                # Returns None

value14 = value.get_as_double(1)                        # Returns 123.456
value15 = value.get_as_double_with_default(2, 0.0)      # Returns 0.0
value16 = value.get_as_nullable_double(2)               # Returns None

value17 = value.get_as_datetime(2)                      # Returns 2018-01-01 00:00:00+00:00
value18 = value.get_as_datetime_with_default(1, date.today())  # Returns (e.g) 2024-06-27 00:00:00+00:00

value19 = value.get_as_string(2)                        # Returns '2018-01-01'
value20 = value.get_as_nullable_string(2)               # Returns '2018-01-01'
value21 = value.get_as_string_with_default(2,'0000-00-00')       # Returns '2018-01-01'

value22 = value.get_as_array(1)                         # Returns ['123.456']
value23 = value.get_as_array_with_default(0, AnyValueArray([0]))       # Returns [1]
value24 = value.get_as_nullable_array(2)                # Returns ['2018-01-01']

valueA = AnyValueArray([1, {'number': "123.456"}, "2018-01-01"])
value25 = valueA.get_as_map(1)                           # Returns {'number': '123.456'}
value25 = valueA.get_as_map_with_default(2, AnyValueMap({'key1': 1})) # Returns {'key1': 1}
value27 = valueA.get_as_nullable_map(2)                  # Returns None

value28 = value.get_as_type(TypeCode.DateTime,2)        # Returns 2018-01-01 00:00:00+00:00
value29 = value.get_as_type_with_default(TypeCode.DateTime, 1, date.today())      # Returns today date
value30 = value.get_as_nullable_type(TypeCode.DateTime,1)                       # Returns None
```

```python
# Replace a value
value = AnyValueArray([1, 2, 3])
value.put(0,"ABC")    # Returns ABC,2,3

# Append a value
my_list = [1,3,4]
value.append(my_list)  # Returns ABC,2,3,1,3,4
```

```python

value = AnyValueArray([1, "123.456", "2018-01-01"])
value.remove(0)    # Returns 123.456,2018-01-01
```

```python
value = AnyValueMap({ 'key1': 1, 'key2': 123.456, 'key3': "ABC", 'key4': date.today() })

```

```python
from pip_services4_commons.data import AnyValueMap
```

```python
from pip_services4_commons.data import AnyValueMap

# Constructor
value = AnyValueMap({ 'key1': 1, 'key2': "123.456", 'key3': "2018-01-01" })    # Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}

# Cloning
value2 = value.clone()    # Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}

# Maps
my_map1 = { 'key1': 1, 'key2': "123.456", 'key3': "2018-01-01" }
my_map2  = {'key4': 2, 'key5': 3, 'key6': 4}

list_of_maps = [my_map1, my_map2]
value = AnyValueMap.from_maps(*list_of_maps)     # Returns {'key1': 1,'key2': '123.456','key3': '2018-01-01','key4': 2,'key5': 3,'key6': 4} 

# Tuples
my_tuple2 = ['key1', 1, 'key2', '123.456', 'key3', "2018-01-01"]
value = AnyValueMap.from_tuples(*my_tuple2)      # Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}

# Array of tuples
my_tuple = ['key1', 1, 'key2', '123.456', 'key3', "2018-01-01"]
value = AnyValueMap.from_tuples_array(my_tuple)   # Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}

# Values
my_value = [1,2]  
value = AnyValueMap.from_value(my_value)         # Returns {'0': 1, '1': 2}
```

```python
from pip_services4_commons.data import AnyValueMap
from pip_services4_commons.convert import TypeCode

value = AnyValueMap({ 'key1': 1, 'key2': "123.456", 'key3': "2018-01-01" })

value1 = value.get_as_boolean("key1")   # Returns: true
value2 = value.get_as_integer("key2")   # Returns: 123
value3 = value.get_as_integer_with_default("key3", 0)   # Returns 0
value4 = value.get_as_float("key2")     # Returns: 123.456
value5 = value.get_as_datetime("key3")  # Returns new Date(2018,0,1)
value9 = value.get_as_string("key1")    # Returns '1'
valueA = AnyValueMap({'key1': 1, 'key2': {'key': "123.456"}, 'key3': "2018-01-01"}) # redact
value6 = valueA.get_as_map("key2")      # Returns {'key': '123.456'}
value7 = value.get_as_nullable_datetime("key2")     # Returns None
value8 = value.get_as_nullable_datetime("key3")     # Returns new Date(2018,0,1)
value10 = value.get_as_object()                     # Returns {'key1': 1, 'key2': '123.456', 'key3': '2018-01-01'}
value11 = value.get_as_type(TypeCode.String, 'key1')     # Returns '1'
value12 = value.get_as_value('key1')
```

```python
value = AnyValueMap({ 'key1': 1, 'key2': "123.456", 'key3': "2018-01-01" })
value.set_as_object('key1', 2)           # Returns {'key1': 2, 'key2': '123.456', 'key3': '2018-01-01'}

```

```python
value = AnyValueMap({ 'key1': 1, 'key2': "123.456", 'key3': "2018-01-01" })
value.remove('key1')     # Returns value = {'key2': '123.456', 'key3': '2018-01-01'}

```
