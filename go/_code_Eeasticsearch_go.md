````
import ellog "github.com/pip-services4/pip-services4-go/pip-services4-elasticsearch-go/log"

````

````
import (
	"context"
	"fmt"

	ellog "github.com/pip-services4/pip-services4-go/pip-services4-elasticsearch-go/log"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

type MyComponentA struct {
	_logger      clog.ILogger
	_console_log bool
}

func NewMyComponentA(logger *ellog.ElasticSearchLogger) *MyComponentA {
	c := &MyComponentA{}
	c._logger = logger

	c._logger.Info(context.Background(), "MyComponentA has been created.")
	return c
}

func (c *MyComponentA) MyMethod() {
	if c._console_log {
		fmt.Println("Hola amigo")
		fmt.Println("Bonjour mon ami")
	}
	c._logger.Info(context.Background(), "Greetings created.")
}

````

````
import cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

logger := ellog.NewElasticSearchLogger()
logger.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 9200,
))
err := logger.Open(context.Background())
if err != nil {
	panic(err)
}
````

```
mycomponent := NewMyComponentA(logger)
for i := 0; i < 10; i++ {
	mycomponent.MyMethod()
}
```

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	ellog "github.com/pip-services4/pip-services4-go/pip-services4-elasticsearch-go/log"
)

func main() {
	logger := ellog.NewElasticSearchLogger()

	logger.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 9200,
	))

	err := logger.Open(context.Background())
	if err != nil {
		panic(err)
	}

	mycomponent := NewMyComponentA(logger)
	for i := 0; i < 10; i++ {
		mycomponent.MyMethod(context.Background())
	}

}

type MyComponentA struct {
	_logger      *ellog.ElasticSearchLogger
	_console_log bool
}

func NewMyComponentA(logger *ellog.ElasticSearchLogger) *MyComponentA {
	c := &MyComponentA{}
	c._logger = logger
	c._console_log = true

	c._logger.Info(context.Background(), "MyComponentA has been created.")
	return c
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c._logger.Configure(ctx, config)
}

func (c *MyComponentA) IsOpen() bool {
	return c._logger.IsOpen()
}

func (c *MyComponentA) Open(ctx context.Context) (err error) {
	return c._logger.Open(ctx)
}

func (c *MyComponentA) Close(ctx context.Context) (err error) {
	return c._logger.Close(ctx)
}

func (c *MyComponentA) MyMethod(ctx context.Context) {
	if c._console_log {
		fmt.Println("Hola amigo")
		fmt.Println("Bonjour mon ami")
	}
	c._logger.Info(context.Background(), "Greetings created.")

	defer c._logger.Info(ctx, "Finally reached.")
}

```
