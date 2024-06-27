```python
from pip_services4_data.validate import Schema, ValueComparisonRule, AndRule
```

```python
# Comparing 1 < x < 10 by using a list of rules
my_rules = [ValueComparisonRule("LTE", 10), ValueComparisonRule("GTE", 1)]
my_schema1 = Schema(rules=my_rules)
```

```python
# Comparing 1 < x < 10 by using the AND rule
my_schema2 = Schema().with_rule(AndRule(ValueComparisonRule("GTE", 1), ValueComparisonRule("LTE", 10)))

```

```python
# Comparing 1 <= x <= 10 by using a list of rules
my_rules = [ValueComparisonRule("LTE", 10), ValueComparisonRule("GTE", 1)]
schema = Schema(rules=my_rules)

validation = schema.validate(0)

# Case 1: bad value
if len(validation) > 0:
    # Case: bad value
    print(validation[0].get_message())
    print(validation[0].get_code())
else:
    # Case: good value
    print("Value within range")

# Case 2: good value  
validate_schema1_2 = schema.validate(5)

if len(validate_schema1_2) > 0:
    # Case: bad value
    print(validate_schema1_2[0].get_message())
    print(validate_schema1_2[0].get_code())
else:
    # Case: good value
    print("Value within range")

```

```python
# NotRule - Case: value different from 1
from pip_services4_data.validate import Schema,NotRule
schema = Schema().with_rule(NotRule(ValueComparisonRule("EQ", 1)))

# Case 1: good value
validation = schema.validate(2)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(1)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
# AndRule - Case:  1<= x <= 10
from pip_services4_data.validate import Schema,AndRule
schema = Schema().with_rule(AndRule(ValueComparisonRule("GTE", 1), ValueComparisonRule("LTE", 10)))

# Case 1: good value
validation = schema.validate(1)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(12)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
# Or rule - Case x < 1 OR x > 10
from pip_services4_data.validate import Schema,OrRule
schema = Schema().with_rule(OrRule(ValueComparisonRule("LT", 1), ValueComparisonRule("GT", 10)))

# Case 1: good value
validation = schema.validate(0)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(5)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
# Included rule - Case: include 1, 2, 3
from pip_services4_data.validate import Schema,IncludedRule
schema = Schema().with_rule(IncludedRule(1, 2, 3))

# Case 1: good value
validation = schema.validate(2)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(10)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
# Excluded rule - Case: excluded values are 1, 2 3
from pip_services4_data.validate import Schema,ExcludedRule
schema = Schema().with_rule(ExcludedRule(1, 2, 3))

# Case 1: good value
validation = schema.validate(10)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(2)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
# Rule At least one exists - Case: field1, field2
from pip_services4_data.validate import Schema,AtLeastOneExistsRule
schema = Schema().with_rule(AtLeastOneExistsRule("field1", "field2"))

# Case 1: good value
validation = schema.validate({ 'field1': 1, 'field2': "A" })
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate({ })
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())
```

```python
from pip_services4_data.validate import Schema, ValueComparisonRule

# Case x < 1
schema = Schema().with_rule(ValueComparisonRule("LT", 1))

# Case 1: good value
validation = schema.validate(0)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

# Case 2: bad value
validation = schema.validate(5)
if len(validation) == 0:
    print("No errors")
else:
    print(validation[0].get_message())
    print(validation[0].get_code())

```
