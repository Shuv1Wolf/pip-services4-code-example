#### Pre-requisites

```
import (
    refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

```

#### Creation

```
import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	pcontroller "github.com/pip-services4/pip-services4-go/pip-services4-prometheus-go/controllers"
	pcount "github.com/pip-services4/pip-services4-go/pip-services4-prometheus-go/count"
)


controller := pcontroller.NewPrometheusMetricsController()
service.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 8080,
))

counters := pcount.NewPrometheusCounters()
counters.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 8081,
))
```

```
import (
        "context"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

references := refer.NewReferencesFromTuples(context.Background(),
	refer.NewDescriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller,
)
```

```
references := refer.NewEmptyReferences()
```

```
references3 := refer.NewReferences(context.Background(), []interface{}{refer.NewDescriptor("pip-services", "context-info", "default", "default", "1.0"), controller})
```

```
references.Find(controller, true)
```

```
references.GetAll()
```

```
references.GetAllLocators()
```

```
references.GetOneOptional(controller)
```

```
references.GetOneRequired(controller)
```

```
references.GetOptional(controller)
```

```
references.GetRequired(controller)
```

```
references2.Put(context.Background(), refer.NewDescriptor("pip-controller", "metrics-controller", "prometheus", "default", "1.0"), controller)
```

```
references2.Remove(context.Background(), controller)
```

```
references.RemoveAll(context.Background(), controller)
```

### Example 1

```
import (
	"context"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ccontext "github.com/pip-services4/pip-services4-go/pip-services4-components-go/context"
)

contextInfo := ccontext.NewContextInfo()
contextInfo.Name = "Test"
contextInfo.Description = "This is a test container"

references := refer.NewReferencesFromTuples(context.Background(),
	refer.NewDescriptor("pip-services", "context-info", "default", "default", "1.0"), contextInfo,
	refer.NewDescriptor("pip-services", "counters", "prometheus", "default", "1.0"), counters,
	refer.NewDescriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller,
)

controller.SetReferences(context.Background(), references)
counters.SetReferences(context.Background(), references)
```

```
type MyComponentA struct {
	counters *pcount.PrometheusCounters

	ConsoleLog bool // console log flag
}

func NewMyComponentA() *MyComponentA {
	c := MyComponentA{
		ConsoleLog: true,
	}
	if c.ConsoleLog {
		fmt.Println("MyComponentA has been created.")
	}
	return &c
}

func (c *MyComponentA) SetReferences(references refer.IReferences) {
	p, err := references.GetOneRequired(
		refer.NewDescriptor("*", "counters", "prometheus", "*", "*"),
	)

	if p != nil && err == nil {
		c.counters = p.(*pcount.PrometheusCounters)
	}
}
```

```
// SetReferences method are sets references to dependent components.
//
//	Parameters:
//		- ctx context.Context	operation context
//		- references  cref.IReferences
//
// references to locate the component dependencies.
func (c *PrometheusCounters) SetReferences(ctx context.Context, references cref.IReferences) {
	c.logger.SetReferences(ctx, references)
	c.connectionResolver.SetReferences(ctx, references)
	ref := references.GetOneOptional(
		cref.NewDescriptor("pip-services", "context-info", "default", "*", "1.0"))
	contextInfo, _ := ref.(*cctx.ContextInfo)
	if contextInfo != nil && c.source == "" {
		c.source = contextInfo.Name
	}
	if contextInfo != nil && c.instance == "" {
		c.instance = contextInfo.ContextId
	}
}
```

```
// SetReferences is sets references to dependent components.
//
//	Parameters:
//		- ctx context.Context	operation context
//		- references cref.IReferences
//
// references to locate the component dependencies.
func (c *PrometheusMetricsController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.RestController.SetReferences(ctx, references)

	resolv := c.DependencyResolver.GetOneOptional("prometheus-counters")
	c.cachedCounters = resolv.(*pcount.PrometheusCounters).CachedCounters
	if c.cachedCounters == nil {
		resolv = c.DependencyResolver.GetOneOptional("cached-counters")
		c.cachedCounters = resolv.(*ccount.CachedCounters)
	}
	ref := references.GetOneOptional(
		cref.NewDescriptor("pip-services", "context-info", "default", "*", "1.0"))
	contextInfo := ref.(*cctx.ContextInfo)

	if contextInfo != nil && c.source == "" {
		c.source = contextInfo.Name
	}
	if contextInfo != nil && c.instance == "" {
		c.instance = contextInfo.ContextId
	}
}
```

### Example 2

```
references := refer.NewReferencesFromTuples(context.Background(),
	refer.NewDescriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller,
	refer.NewDescriptor("pip-services", "metrics-controller", "prometheus", "default", "2.0"), controller2,
	refer.NewDescriptor("pip-services", "counters", "prometheus", "default", "1.0"), counters,
	refer.NewDescriptor("pip-services", "context-info", "default", "*", "1.0"), cinfo.NewContextInfo(),
	refer.NewDescriptor("pip-services", "logger", "console", "default", "1.0"), clog.NewConsoleLogger(),
)
```

```
references.Find(refer.NewDescriptor("*", "metrics-controller", "*", "*", "*"), true)

```

```
references.GetOneOptional(refer.NewDescriptor("*", "metrics-controller", "*", "*", "2.0"))

```

```
references.GetRequired(refer.NewDescriptor("pip-services", "*", "*", "*", "*"))

```

```
reference := references.GetOneOptional(controller)

```
