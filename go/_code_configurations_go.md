```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)
```

```
// Example: create a ConfigParams object containing {'section1.key1': 'AAA', 'section1.key2': '123', 'section2.key1': 'True'}

// Constructor

myDict := map[string]string{"section1.key1": "AAA", "section1.key2": "123", "section2.key1": "true"}
config1 := conf.NewConfigParams(myDict)

// Tuple
config2 := conf.NewConfigParamsFromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true)

// String
config3 := conf.NewConfigParamsFromString("section1.key1=AAA;section1.key2=123;section2.key1=true")

// Object containing key:value pairs
myDict = map[string]string{"section1.key1": "AAA", "section1.key2": "123", "section2.key1": "true"}
config4 := conf.NewConfigParamsFromValue(myDict)
```

```
// Add a new section
config := conf.NewConfigParamsFromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true)
config.AddSection("section3", conf.NewConfigParamsFromTuples("key1", "ABCDE"))
```

```
// Get section
config := conf.NewConfigParamsFromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true)
section1 := config.GetSection("section1") // Returns {'key1': 'AAA', 'key2': '123'}
```

```
config := conf.NewConfigParamsFromTuples("section1.key1", "AAA", "section1.key2", 123, "section2.key1", true)
config.GetSectionNames() // Returns ['section1', 'section2']
```

```
import (
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

func main() {
	additionalConfig1 := conf.NewConfigParamsFromTuples(
		"my_store1.user", "jdoe",
		"my_store1.password", "pass123",
		"my_store1.pin", "321",
	)

	additionalConfig2 := conf.NewConfigParamsFromTuples(
		"my_store2.user", "David",
		"my_store2.password", "another_pass",
		"my_store2.pin", "0000",
	)

	additionalConfig3 := conf.NewConfigParamsFromTuples(
		"param1", "value1",
	)

	config := conf.NewConfigParamsFromMaps(additionalConfig1.Value(), additionalConfig2.Value(), additionalConfig3.Value())

	fmt.Println(config)
}
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)
```

```
// name
config := conf.NewConfigParamsFromTuples("name", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123)
name := conf.NameResolver.Resolve(config) // Returns myservice:connector:aws:connector1:1.0

// id
config = conf.NewConfigParamsFromTuples("id", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123)
id := conf.NameResolver.Resolve(config) // Returns myservice:connector:aws:connector1:1.0

// If name cannot be determined

config = conf.NewConfigParamsFromTuples("param1", "ABC", "param2", 123)
name = conf.NameResolver.Resolve(config)                            // Returns ""
name = conf.NameResolver.ResolveWithDefault(config, "default name") //Returns "default name"

// name and id
config = conf.NewConfigParamsFromTuples("name", "my_name", "id", "my id",
	"param1", "ABC",
	"param2", 123)
result := conf.NameResolver.Resolve(config) // Returns "my_name"

// descriptor
// Note: A descriptor class has the following parameters: "group", "type", "kind", "name", "version"
//       Name resolver will extract the value of the "name" parameter.

config = conf.NewConfigParamsFromTuples("descriptor", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123)
name = conf.NameResolver.Resolve(config) // Returns connector1
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)
```

```
// ConfigParams objec contains an option section

config := conf.NewConfigParamsFromTuples(
	"section1.key1", "AAA",
	"section1.key2", 123,
	"options.param1", "ABC",
	"options.param2", 234)

options := conf.OptionsResolver.Resolve(config) // Returns {'param1': 'ABC', 'param2': '234'}

// ConfigParams object doesn't contain an "options" section
config = conf.NewConfigParamsFromTuples(
	"section1.key1", "AAA",
	"section1.key2", 123,
)
options = conf.OptionsResolver.Resolve(config) // Returns {}

// ConfigParams object doesn't contain an "options" section, and the config_as_default parameter is set to True.
config = conf.NewConfigParamsFromTuples(
	"section1.key1", "AAA",
	"section1.key2", 123,
)
options = conf.OptionsResolver.ResolveWithDefault(config) // Returns {'section1.key1': 'AAA', 'section1.key2': '123'}
```
