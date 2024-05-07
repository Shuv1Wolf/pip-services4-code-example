```
import (
	validate "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
)
```

```
// Comparing 1 < x < 10 by using a list of rules
myRules := []validate.IValidationRule{validate.NewValueComparisonRule("LTE", 10), validate.NewValueComparisonRule("GTE", 1)}
mySchema := validate.NewSchemaWithRules(false, myRules)
```

```
// Comparing 1 < x < 10 by using the AND rule
mySchema2 := validate.NewSchema().WithRule(
	validate.NewAndRule(
		validate.NewValueComparisonRule("GTE", 1),
		validate.NewValueComparisonRule("LTE", 10),
))
```

```
// Comparing 1 <= x <= 10 by using a list of rules
myRules := []validate.IValidationRule{
	validate.NewValueComparisonRule("LTE", 10),
	validate.NewValueComparisonRule("GTE", 1),
}

mySchema := validate.NewSchemaWithRules(false, myRules)

// Case 1: bad value
validation := mySchema.Validate(0)

if len(validation) > 0 {
	// Case: bad value
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
} else {
	// Case: good value
	fmt.Println("Value within range")
}

// Case 2: good value
validation = mySchema.Validate(5)

if len(validation) > 0 {
	// Case: bad value
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
} else {
	// Case: good value
	fmt.Println("Value within range")
}
```

```
// NotRule - Case: value different from 1
schema := validate.NewSchema().WithRule(validate.NewNotRule(validate.NewValueComparisonRule("EQ", 1)))

// Case 1: good value
validation := schema.Validate(2)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(1)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```

```

// AndRule - Case:  1<= x <= 10
schema := validate.NewSchema().WithRule(validate.NewAndRule(
	validate.NewValueComparisonRule("GTE", 1),
	validate.NewValueComparisonRule("LTE", 10),
))

// Case 1: good value
validation := schema.Validate(1)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(12)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```

```
// Or rule - Case x < 1 OR x > 10
schema := validate.NewSchema().WithRule(validate.NewOrRule(
	validate.NewValueComparisonRule("LT", 1),
	validate.NewValueComparisonRule("GT", 10),
))

// Case 1: good value
validation := schema.Validate(0)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(5)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```

```
// Included rule - Case: include 1, 2, 3
schema := validate.NewSchema().WithRule(validate.NewIncludedRule(1, 2, 3))

// Case 1: good value
validation := schema.Validate(2)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(10)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```

```
// Excluded rule - Case: excluded values are 1, 2 3
schema := validate.NewSchema().WithRule(validate.NewExcludedRule(1, 2, 3))

// Case 1: good value
validation := schema.Validate(10)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(2)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```

```

// Rule At least one exists - Case: field1, field2
schema := validate.NewSchema().WithRule(validate.NewAtLeastOneExistsRule("field1", "field2"))

// Case 1: good value
validation := schema.Validate(map[string]interface{}{"field1": 1, "field2": "A"})

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(map[string]interface{}{})

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

```

```
schema := validate.NewSchema().WithRule(validate.NewValueComparisonRule("LT", 1))

// Case 1: good value
validation := schema.Validate(0)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}

// Case 2: bad value
validation = schema.Validate(5)

if len(validation) == 0 {
	fmt.Println("No errors")
} else {
	fmt.Println(validation[0].Message())
	fmt.Println(validation[0].Code())
}
```
