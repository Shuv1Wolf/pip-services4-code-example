```
import (
	"context"
	"fmt"

	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
)

type MyComponent struct {
	_consoleLog bool
	_counters   ccount.ICounters
}

func NewMyComponentA(counters ccount.ICounters) *MyComponent {
	c := &MyComponent{}
	c._counters = counters

	if c._consoleLog {
		fmt.Println("MyComponent has been created.")
	}

	return c
}

func (c *MyComponent) Mymethod() {
	c._counters.Increment(context.Background(), "mycomponent.mymethod.calls", 1)
	timing := c._counters.BeginTiming(context.Background(), "mycomponent.mymethod.exec_time")

	defer timing.EndTiming(context.Background())
	if c._consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Bonjour mon ami")
	}
}
```

```
import (
	"context"
	"fmt"

	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
)

func main() {
	countersCached := NewMyCachedCounters()
	mycomponentCached := NewMyComponentA(countersCached)

	countExec := 2

	for i := 0; i < countExec; i++ {
		mycomponentCached.Mymethod()
	}

	resultCached := countersCached.GetAll()

	fmt.Println("Metrics")

	for _, res := range resultCached {
		fmt.Printf("Count: %d\n", res.Count())
		fmt.Printf("Min: %02f\n", res.Min())
		fmt.Printf("Max: %02f\n", res.Max())
		fmt.Printf("Average: %02f\n", res.Average())
		fmt.Printf("Time: %s\n", res.Time())
		fmt.Printf("Name: %s\n", res.Name())
		fmt.Printf("Type: %d\n", res.Type())
		fmt.Printf("-----------------")
	}
}

type MyCachedCounters struct {
	*ccount.CachedCounters
}

func NewMyCachedCounters() *MyCachedCounters {
	c := &MyCachedCounters{}
	c.CachedCounters = ccount.InheritCacheCounters(c)
	return c
}

func (c *MyCachedCounters) Save(ctx context.Context, counters []ccount.Counter) error {
	fmt.Println("Saving " + counters[0].Name + " and " + counters[1].Name)
	return nil
}

```

```
import (
	"context"
	"fmt"

	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

func main() {
	counters := ccount.NewLogCounters()
	counters.SetReferences(context.Background(), cref.NewReferencesFromTuples(context.Background(),
		cref.NewDescriptor("pip-services", "logger", "console", "default", "1.0"),
		clog.NewConsoleLogger(),
	))

	mycomponentLog := NewMyComponentA(counters)

	countExec := 2

	for i := 0; i < countExec; i++ {
		mycomponentLog.Mymethod()
	}

	resultLog := counters.GetAll()

	fmt.Println("Metrics")

	for _, res := range resultLog {
		fmt.Printf("Count: %d\n", res.Count())
		fmt.Printf("Min: %02f\n", res.Min())
		fmt.Printf("Max: %02f\n", res.Max())
		fmt.Printf("Average: %02f\n", res.Average())
		fmt.Printf("Time: %s\n", res.Time())
		fmt.Printf("Name: %s\n", res.Name())
		fmt.Printf("Type: %d\n", res.Type())
		fmt.Printf("-----------------")
	}
}

```

```
import (
	"context"
	"fmt"

	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
)

func main() {
	countersNull := ccount.NewNullCounters()

	mycomponentNull := NewMyComponent(countersNull)

	countExec := 2

	for i := 0; i < countExec; i++ {
		mycomponentNull.Mymethod(context.Background())
	}
}

type MyComponent struct {
	_consoleLog bool
	_counters   ccount.ICounters
}

func NewMyComponent(counters ccount.ICounters) *MyComponent {
	c := &MyComponent{}
	c._counters = counters

	if c._consoleLog {
		fmt.Println("MyComponent has been created.")
	}

	return c
}

func (c *MyComponent) Mymethod(ctx context.Context) {
	c._counters.Increment(context.Background(), "mycomponent.mymethod.calls", 1)
	timing := c._counters.BeginTiming(context.Background(), "mycomponent.mymethod.exec_time")

	defer timing.EndTiming(ctx)
	if c._consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Bonjour mon ami")
	}
}
```

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
	pcount "github.com/pip-services4/pip-services4-go/pip-services4-prometheus-go/count"
)

func main() {
	countersLog := ccount.NewLogCounters()

	countersProm := pcount.NewPrometheusCounters()
	countersProm.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 8080,
	))

	myComponent := NewMyComponent()

	myComponent.SetReferences(context.Background(), cref.NewReferencesFromTuples(context.Background(),
		cref.NewDescriptor("pip-services", "counters", "logger", "default3", "1.0"), countersLog,
		cref.NewDescriptor("pip-services", "counters", "prometheus", "default4", "1.0"), countersProm,
		cref.NewDescriptor("pip-services", "logger", "cached", "default2", "1.0"), NewMyCachedLogger(),
	))

	err := countersProm.Open(context.Background())
	if err != nil {
		panic(err)
	}

	countExec := 2

	for i := 0; i < countExec; i++ {
		myComponent.Mymethod(context.Background())
	}

	results := countersLog.GetAll()
	counters := make([]ccount.Counter, 0)

	for _, val := range results {
		counters = append(counters, val.GetCounter())
	}

	fmt.Println("Metrics to logger")
	PrintResults(counters)

	results = countersProm.GetAll()

	for _, val := range results {
		counters = append(counters, val.GetCounter())
	}

	fmt.Println("Metrics to Prometheus")
	PrintResults(counters)
}

func PrintResults(results []ccount.Counter) {
	for _, res := range results {
		fmt.Printf("Count: %d\n", res.Count)
		fmt.Printf("Min: %02f\n", res.Min)
		fmt.Printf("Max: %02f\n", res.Max)
		fmt.Printf("Average: %02f\n", res.Average)
		fmt.Printf("Time: %s\n", res.Time)
		fmt.Printf("Name: %s\n", res.Name)
		fmt.Printf("Type: %d\n", res.Type)
		fmt.Printf("-----------------")
	}
}

type MyComponent struct {
	_consoleLog bool
	counters    *ccount.CompositeCounters
}

func NewMyComponent() *MyComponent {
	c := &MyComponent{
		counters: ccount.NewCompositeCounters(),
	}

	if c._consoleLog {
		fmt.Println("MyComponent has been created.")
	}

	return c
}

func (c *MyComponent) SetReferences(ctx context.Context, references cref.IReferences) {
	c.counters.SetReferences(ctx, references)
}

func (c *MyComponent) Mymethod(ctx context.Context) {
	c.counters.Increment(context.Background(), "mycomponent.mymethod.calls", 1)
	timing := c.counters.BeginTiming(context.Background(), "mycomponent.mymethod.exec_time")

	defer timing.EndTiming(ctx)
	if c._consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Bonjour mon ami")
	}
}

type MyCachedLogger struct {
	*clog.CachedLogger
}

func NewMyCachedLogger() *MyCachedLogger {
	c := &MyCachedLogger{}
	c.CachedLogger = clog.InheritCachedLogger(c)
	return c
}

func (c *MyCachedLogger) Save(ctx context.Context, messages []clog.LogMessage) error {
	fmt.Println("Saving somewhere")
	return nil
}

```
