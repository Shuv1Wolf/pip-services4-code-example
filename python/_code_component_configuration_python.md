```python
class IConfigurable(ABC):

    def configure(self, config: ConfigParams):
        raise NotImplementedError('Method from interface definition')


```

```python
from pip_services4_components.config import ConfigParams

config = ConfigParams.from_tuples(
  "param1", 123,
  "param2", "2020-01-01T11:00:00.0Z"
)
```

```
param1 = config.get_as_integer("param1")
param2 = config.get_as_datetime_with_default("param2", datetime.datetime.now())
```

```
from pip_services4_components.config import ConfigParams

config_with_sections = ConfigParams.from_tuples(
  "param1", 123,
  "options.param1", "ABC",
  "options.param2", "XYZ"
)
options = config_with_sections.get_section("options")
```

```
default_config = ConfigParams.from_tuples(
  "param1", 1,
  "param2", "Default Value"
)
config = config.set_defaults(default_config)

```

```
another_config = ConfigParams.from_string("param1=123;param2=ABC")
```

```
class DataController(IConfigurable):
   __max_page_size: int = 5


   def configure(self, config: ConfigParams):
    self.__max_page_size = config.get_as_integer_with_default('max_page_size', self.__max_page_size)
   
   def get_data(self, filter: FilterParams, paging: PagingParams) -> DataPage: 
    paging.take = min(paging.take, self.__max_page_size)   
        # Get data using max page size constraint.
```

```
component = DataController()
config = ConfigParams.from_tuples("max_page_size", 100)
component.configure(config)


```

```
...
# Controller
- descriptor: "beacons:controller:default:default:1.0"
  max_page_size: 10
...


```

```
config = ConfigParams.from_tuples(
	"descriptor", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123
)
name = NameResolver.resolve(config) # Result: connector1
```

```
config = ConfigParams.from_tuples(
    ...
	"options.param1", "ABC",
	"options.param2", 123
)
options = OptionsResolver.resolve(config)   # Result: param1=ABC;param2=123


```

```
class IConfigReader(ABC):

    def read_config_(self, parameters: ConfigParams) -> ConfigParams:
        raise NotImplementedError('Method from interface definition')


```

```
config = ConfigParams.from_tuples(
	"connection.host", "localhost",
	"connection.port", "8080"
)
config_reader = MemoryConfigReader()
config_reader.configure(config)
parameters = ConfigParams.from_value(sys.argv)
config_reader.read_config(parameters) # Result: connection.host=localhost;connection.port=8080


```

```
{ "key1": "{{KEY1_VALUE}}", "key2": "{{KEY2_VALUE}}" }

```

```
from pip_services4_components.config import ConfigParams
from pip_services4_config.config import JsonConfigReader
from pip_services4_components.context import Context

context_data = {
  "traceId": "123",
}
my_context = Context(context_data)

configReader = JsonConfigReader("config.json")
parameters = ConfigParams.from_tuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC")
configReader.read_config_(my_context, parameters)    # Result: key1=123;key2=ABC

```

```

key1: "1234"
key2: "ABCD"
```

```
configReader = YamlConfigReader("config.yml")
parameters = ConfigParams.from_tuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC")
configReader.read_config_(my_context, parameters)    # Result: key1=1234;key2=ABCD


```
