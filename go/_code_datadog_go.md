```
import (
    dcount "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/count"
)

```

```
import (
	"context"
	"fmt"

	dcount "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/count"
)

type MyComponentA struct {
	consoleLog bool
	counters   *dcount.DataDogCounters
}

func NewMyComponentA(counters *dcount.DataDogCounters) *MyComponentA {
	c := &MyComponentA{
		consoleLog: true,
		counters:   counters,
	}

	if c.consoleLog {
		fmt.Println("MyComponentA has been created.")
	}
	return c
}

func (c *MyComponentA) MyMethod(ctx context.Context) {
	c.counters.Increment(ctx, "mycomponent.mymethod.calls", 1)
	timing := c.counters.BeginTiming(ctx, "mycomponent.mymethod.exec_time")

	defer timing.EndTiming(ctx)

	if c.consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Hola amigoBonjour mon ami")
	}

	c.counters.Dump(ctx)
}
```

```
import conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

counters := dcount.NewDataDogCounters()
counters.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"credential.access_key", "e1be2e70036b8f05f2777f5f038fc222",
))

_ = counters.Open(context.Background())
```

```
mycomponent := NewMyComponentA(counters)

for i := 1; i < 5; i++ {
	mycomponent.MyMethod(context.Background())
}
```

```
import (
	"context"
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	dcount "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/count"
)

type MyComponentA struct {
	consoleLog bool
	counters   *dcount.DataDogCounters
}

func NewMyComponentA(counters *dcount.DataDogCounters) *MyComponentA {
	c := &MyComponentA{
		consoleLog: true,
		counters:   counters,
	}

	if c.consoleLog {
		fmt.Println("MyComponentA has been created.")
	}
	return c
}

func (c *MyComponentA) Configure(ctx context.Context, config *conf.ConfigParams) {
	c.counters.Configure(ctx, config)
}

func (c *MyComponentA) GetCounters() *dcount.DataDogCounters {
	return c.counters
}

func (c *MyComponentA) IsOpen() bool {
	return c.counters.IsOpen()
}

func (c *MyComponentA) Open(ctx context.Context) error {
	return c.counters.Open(ctx)
}

func (c *MyComponentA) Close(ctx context.Context) error {
	return c.counters.Close(ctx)
}

func (c *MyComponentA) MyMethod(ctx context.Context) {
	c.counters.Increment(ctx, "mycomponent.mymethod.calls", 1)
	timing := c.counters.BeginTiming(ctx, "mycomponent.mymethod.exec_time")

	defer timing.EndTiming(ctx)

	if c.consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Hola amigoBonjour mon ami")
	}

	c.counters.Dump(ctx)
}
```

````
import (
	dlog "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/log"
)
````

````
type MyComponentA struct {
	consoleLog bool
	logger     *dlog.DataDogLogger
}

func NewMyComponentA(logger *dlog.DataDogLogger) *MyComponentA {
	c := &MyComponentA{
		consoleLog: true,
		logger:     logger,
	}

	if c.consoleLog {
		logger.Info(context.Background(), "MyComponentA has been created.")
	}
	return c
}

func (c *MyComponentA) MyMethod(ctx context.Context) {
	defer c.logger.Info(ctx, "123", "Finally reached.")

	if c.consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Hola amigoBonjour mon ami")
		c.logger.Info(ctx, "123", "Greetings created.")
	}
}
````

```
logger := dlog.NewDataDogLogger()
logger.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"credential.access_key", "e1be2e70036b8f05f2777f5f038fc222",
))
_ = logger.Open(context.Background())
```

```
mycomponent := NewMyComponentA(logger)

for i := 1; i < 5; i++ {
	mycomponent.MyMethod(context.Background())
}
```

```
import (
	"context"
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	dlog "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/log"
)

type MyComponentA struct {
	consoleLog bool
	logger     *dlog.DataDogLogger
}

func NewMyComponentA(logger *dlog.DataDogLogger) *MyComponentA {
	c := &MyComponentA{
		consoleLog: true,
		logger:     logger,
	}

	if c.consoleLog {
		logger.Info(context.Background(), "123", "MyComponentA has been created.")
	}
	return c
}

func (c *MyComponentA) Configure(ctx context.Context, config *conf.ConfigParams) {
	c.logger.Configure(ctx, config)
}

func (c *MyComponentA) GetCounters() *dlog.DataDogLogger {
	return c.logger
}

func (c *MyComponentA) IsOpen() bool {
	return c.logger.IsOpen()
}

func (c *MyComponentA) Open(ctx context.Context) error {
	return c.logger.Open(ctx)
}

func (c *MyComponentA) Close(ctx context.Context) error {
	return c.logger.Close(ctx)
}

func (c *MyComponentA) MyMethod(ctx context.Context) {
	defer c.logger.Info(ctx, "Finally reached.")

	if c.consoleLog {
		fmt.Println("Hola amigo")
		fmt.Println("Hola amigoBonjour mon ami")
		c.logger.Info(ctx, "Greetings created.")
	}
}
```
