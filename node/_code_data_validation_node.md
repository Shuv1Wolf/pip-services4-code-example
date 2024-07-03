```
import { AndRule, Schema, ValueComparisonRule } from "pip-services4-data-node"
```

```
// Comparing 1 < x < 10 by using a list of rules
let myRules = [new ValueComparisonRule("LTE", 10), new ValueComparisonRule("GTE", 1)];
let mySchema1 = new Schema(false, myRules);



```

```
// Comparing 1 < x < 10 by using the AND rule
let mySchema2 = new Schema().withRule(new AndRule(new ValueComparisonRule("GTE", 1), new ValueComparisonRule("LTE", 10)));

```

```
// Comparing 1 <= x <= 10 by using a list of rules
let myRules = [new ValueComparisonRule("LTE", 10), new ValueComparisonRule("GTE", 1) ];
let schema = new Schema(false, myRules);

// Case 1: bad value
let validation = schema.validate(0);

if (validation.length > 0) {
    // Case: bad value
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
} else {
    // Case: good value
    console.log("Value within range");
}


// Case 2: good value   
validation = schema.validate(5);

if (validation.length > 0) {
    // Case: bad value
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}
else {
    // Case: good value
    console.log("Value within range");
}


```

```
// NotRule - Case: value different from 1
let schema = new Schema().withRule(new NotRule(new ValueComparisonRule("EQ", 1)));

// Case 1: good value
let validation = schema.validate(2);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(1);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
// AndRule - Case:  1<= x <= 10
let schema = new Schema().withRule(new AndRule(new ValueComparisonRule("GTE", 1), new ValueComparisonRule("LTE", 10)));

// Case 1: good value
let validation = schema.validate(1);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(12);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
// Or rule - Case x < 1 OR x > 10
let schema = new Schema().withRule(new OrRule(new ValueComparisonRule("LT", 1), new ValueComparisonRule("GT", 10)));

// Case 1: good value
let validation = schema.validate(0);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(5);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
// Included rule - Case: include 1, 2, 3
let schema = new Schema().withRule(new IncludedRule(1, 2, 3));

// Case 1: good value
let validation = schema.validate(2);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(10);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
// Excluded rule - Case: excluded values are 1, 2 3
let schema = new Schema().withRule(new ExcludedRule(1, 2, 3));

// Case 1: good value
let validation = schema.validate(10);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(2);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
// Rule At least one exists - Case: field1, field2
let schema = new Schema().withRule(new AtLeastOneExistsRule("field1", "field2"));

// Case 1: good value
let validation = schema.validate({ "field1": 1 , "field2": "A" });

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(2);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```

```
var schema = new Schema().withRule(new ValueComparisonRule("LT", 1));

// Case 1: good value
var validation = schema.validate(0);

if (validation.length == 0) {
    console.log("No errors");
} else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


// Case 2: bad value
validation = schema.validate(5);

if (validation.length == 0) {
    console.log("No errors");
}
else {
    console.log(validation[0].getMessage());
    console.log(validation[0].getCode());
}


```
