```
import { ConfigParams } from "pip-services4-components-node"
```

```
// Example: create a ConfigParams object containing {'section1.key1': 'AAA', 'section1.key2': '123', 'section2.key1': 'True'}

// Constructor

let myDict = { "section1.key1": "AAA", "section1.key2": 123, "section2.key1": true };
let config1 = new ConfigParams(myDict);

// Tuple
let config2 = ConfigParams.fromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true);

// String
let config3 = ConfigParams.fromString("section1.key1=AAA;section1.key2=123;section2.key1=True");

// Object containing key:value pairs
myDict = { "section1.key1": "AAA", "section1.key2": 123, "section2.key1": true };
let config4 = ConfigParams.fromValue(myDict);
```

```
// Add a new section 
let config = ConfigParams.fromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true);
config.addSection("section3", ConfigParams.fromTuples("key1", "ABCDE"));



```

```
let config = ConfigParams.fromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true);
let section1 = config.getSection("section1");      // Returns {'key1': 'AAA', 'key2': '123'} 


```

```
let config = ConfigParams.fromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true);
config.getSectionNames();      // Returns ['section1', 'section2']


```

```
let additionalConfig1 = ConfigParams.fromTuples(
    "my_store1.user", "jdoe",
    "my_store1.password", "pass123",
    "my_store1.pin", "321"
);

let additionalConfig2 = ConfigParams.fromTuples(
    "my_store2.user", "David",
    "my_store2.password", "another_pass",
    "my_store2.pin", "0000"
);

let additionalConfig3 = ConfigParams.fromTuples(
    "param1", "value1"
);
  
let config = ConfigParams.mergeConfigs(additionalConfig1, additionalConfig2, additionalConfig3);
console.log(config);


```

```
import { NameResolver } from "pip-services4-components-node"
```

```
// name
let config = ConfigParams.fromTuples("name", "myservice:connector:aws:connector1:1.0",
    "param1", "ABC",
    "param2", 123);
let name = NameResolver.resolve(config); // Returns myservice:connector:aws:connector1:1.0

// id
config = ConfigParams.fromTuples("id", "myservice:connector:aws:connector1:1.0",
    "param1", "ABC",
    "param2", 123);
let id = NameResolver.resolve(config);  // Returns myservice:connector:aws:connector1:1.0

// If name cannot be determined

config = ConfigParams.fromTuples("param1", "ABC", "param2", 123);
name = NameResolver.resolve(config); // Returns null
name = NameResolver.resolve(config, "default name"); //Returns "default name"

// name and id
config = ConfigParams.fromTuples("name", "my_name", "id", "my id",
    "param1", "ABC",
    "param2", 123);
let result = NameResolver.resolve(config); // Returns "my_name"

// descriptor
// Note: A descriptor class has the following parameters: "group", "type", "kind", "name", "version"
//       Name resolver will extract the value of the "name" parameter.

config = ConfigParams.fromTuples("descriptor", "myservice:connector:aws:connector1:1.0",
    "param1", "ABC",
    "param2", 123);
name = NameResolver.resolve(config); // Returns connector1


```

```
import { OptionResolver } from "pip-services4-components-node"
```

```
// ConfigParams objec contains an option section

var config = ConfigParams.fromTuples(
    "section1.key1", "AAA",
    "section1.key2", 123,
    "options.param1", "ABC",
    "options.param2", 234);

var options = OptionResolver.resolve(config); // Returns {'param1': 'ABC', 'param2': '234'}

// ConfigParams object doesn't contain an "options" section
config = ConfigParams.fromTuples(
    "section1.key1", "AAA",
    "section1.key2", 123
);

options = OptionResolver.resolve(config, false); // Returns null

// ConfigParams object doesn't contain an "options" section, and the config_as_default parameter is set to True.
config = ConfigParams.fromTuples(
    "section1.key1", "AAA",
    "section1.key2", 123
);

options = OptionResolver.resolve(config, true); // Returns {'section1.key1': 'AAA', 'section1.key2': '123'}


```
