```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type IConfigurable interface {
	Configure(ctx context.Context, config *conf.ConfigParams)
}
```

```
import (
	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

config := cconfig.NewConfigParamsFromTuples(
	"param1", 123,
	"param2", "2020-01-01T11:00:00.0Z",
)
```

```
param1 := config.GetAsInteger("param1")
param2 := config.GetAsDateTimeWithDefault("param2", time.Now())
```

```
configWithSections := cconfig.NewConfigParamsFromTuples(
	"param1", 123,
	"options.param1", "ABC",
	"options.param2", "XYZ",
)
options := configWithSections.GetSection("options")
```

```
defaultConfig := cconfig.NewConfigParamsFromTuples(
	"param1", 1,
	"param2", "Default Value",
)
config = config.SetDefaults(defaultConfig)
```

```
anotherConfig := cconfig.NewConfigParamsFromString("param1=123;param2=ABC")

```

```
import (
	"context"
	"math"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type DataService struct {
	_maxPageSize int
}

func NewDataService() *DataService {
	return &DataService{
		_maxPageSize: 5,
	}
}

func (c *DataService) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._maxPageSize = config.GetAsIntegerWithDefault("max_page_size", c._maxPageSize)
}

func (c *DataService) GetData(ctx context.Context, correlationId string, filter *cquery.FilterParams, paging *cquery.PagingParams) (page *cquery.DataPage, err error) {
	paging.Take = int64(math.Min(float64(paging.Take), float64(c._maxPageSize)))
	// Get data using max page size constraint.
	return page, err
}
```

```
component := configurationexample.NewDataService()
config := cconfig.NewConfigParamsFromTuples("max_page_size", 100)
component.Configure(context.Background(), config)
```

```
...
# Service
- descriptor: "beacons:service:default:default:1.0"
  max_page_size: 10
...
```

```
config := cconfig.NewConfigParamsFromTuples(
	"descriptor", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123,
)
name := cconfig.NameResolver.Resolve(config) // Result: connector1
```

```
config := cconfig.NewConfigParamsFromTuples(
	...
	"options.param1", "ABC",
	"options.param2", 123,
)
options := cconfig.OptionsResolver.Resolve(config) // Result: param1=ABC;param2=123
```

````
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type IConfigReader interface {
	ReadConfig(ctx context.Context, correlationId string, parameters *cconfig.ConfigParams) (*cconfig.ConfigParams, error)
}
````

```
import (
	"context"
	"os"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	creader "github.com/pip-services4/pip-services4-go/pip-services4-config-go/config"
)


config := cconfig.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", "8080",
)

configReader := creader.NewEmptyMemoryConfigReader()
configReader.Configure(context.Background(), config)
parameters := cconfig.NewConfigParamsFromValue(os.Args)
configReader.ReadConfig(context.Background(), parameters) // Result: connection.host=localhost;connection.port=8080

```

```
{ "key1": "{{KEY1_VALUE}}", "key2": "{{KEY2_VALUE}}" }
```

```
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	creader "github.com/pip-services4/pip-services4-go/pip-services4-config-go/config"
)

configReader := creader.NewJsonConfigReader("config.json")
parameters := cconfig.NewConfigParamsFromTuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC")
configReader.ReadConfig(context.Background(), parameters) // Result: key1=123;key2=ABC

```

```
key1: "1234"
key2: "ABCD"
```

```
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	creader "github.com/pip-services4/pip-services4-go/pip-services4-config-go/config"
)

configReader := creader.NewYamlConfigReader("config.yml")
parameters := cconfig.NewConfigParamsFromTuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC")
configReader.ReadConfig(context.Background(), parameters) // Result: key1=123;key2=ABC

```
