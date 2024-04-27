### Design-time Configuration

```
import (
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

```

```
config := cconf.NewConfigParamsFromTuples(
	"param1", "XYZ",
	"param2", 345,
)

component.Configure(context.Background(), config)
```

```
/// Also, often components can have hard-coded presets.
/// The ConfigParams class has methods that allow to easily use them as defaults:

type MyComponent struct { // Implements IConfigurable
	param1 string
	param2 int
}

func NewMyComponent() MyComponent {
	return MyComponent{
		param1: "ABC",
		param2: 123,
	}
}

func (c *MyComponent) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.param1 = config.GetAsStringWithDefault("param1", c.param1)
	c.param2 = config.GetAsIntegerWithDefault("param2", c.param2)
}
```

### Deployment-time Configuration

```
export MYCOMPONENT_ENABLED=true
export PARAM1=XYZ
go run ./bin/main.go -p PARAM2=345

```

```
{{#if MYCOMPONENT_ENABLED}}
descriptor: myservice:mycomponent:default:default:1.0
param1: {{PARAM1}}
param2: {{PARAM2}}
{{/if}}

```
