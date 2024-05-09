```
import (
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

// Logger setting
logger := *clog.NewConsoleLogger()
logger.SetLevel(5)
```

```
func MyTask() {
	// create an artificial problem
	err := errors.New("Logging demo. Error created")
	logger.Error(context.Background(), err, "Error created.")
}
```

```
import (
	"context"
	"errors"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	crun "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

type MyComponentB struct {
	crefer.IReferences
	crefer.IUnreferenceable
	cconfig.IConfigurable
	crun.IOpenable
	crun.ICleanable

	_status string
	_param1 string
	_param2 int
	_opened bool
	_logger clog.ILogger

	dummy_variable interface{}
}

// Creates a new instance of the component.
func NewMyComponentB() *MyComponentB {
	component := &MyComponentB{
		_status: "Created",
		_param1: "ABC2",
		_param2: 456,
		_opened: false,
		_logger: clog.NewConsoleLogger(),

		dummy_variable: "resource variable",
	}

	component._logger.SetLevel(5)
	component._logger.Info(context.Background(), "MyComponentB has been created.")

	return component
}

func (c *MyComponentB) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._param1 = config.GetAsStringWithDefault("param1", c._param1)
	c._param2 = config.GetAsIntegerWithDefault("param2", c._param2)

	c._logger.Info(ctx, "MyComponentB has been configured.")
}

func (c *MyComponentB) SetReferences(ctx context.Context, references *crefer.References) {
	// pass
}

func (c *MyComponentB) isOpen() bool {
	// pass
	return true
}

func (c *MyComponentB) Open(ctx context.Context) {
	// pass
}

func (c *MyComponentB) Close(ctx context.Context) {
	// pass
}

func (c *MyComponentB) MyTask(ctx context.Context) {
	// pass
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentB) UnsetReferences(ctx context.Context) {
	// pass
}

// Clears component state.
// - correlationId: (optional) transaction id to trace execution through call chain.
func (c *MyComponentB) Clear(ctx context.Context) {
	// pass
}

type MyComponentA struct {
	crefer.IReferences
	crefer.IUnreferenceable
	cconfig.IConfigurable
	crun.IOpenable
	crun.ICleanable

	_logger clog.ILogger

	_status string
	_param1 string
	_param2 int
	_opened bool

	dummy_variable interface{}

	_another_component interface{}
}

// Creates a new instance of the component.
func NewMyComponentA() *MyComponentA {
	component := &MyComponentA{
		_status: "Created",
		_param1: "ABC2",
		_param2: 456,
		_opened: false,
		_logger: clog.NewConsoleLogger(),

		dummy_variable: "dummy_variable setted",
	}

	component._logger.SetLevel(5)
	component._logger.Info(context.Background(), "MyComponentA has been created.")

	return component
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._param1 = config.GetAsStringWithDefault("param1", c._param1)
	c._param2 = config.GetAsIntegerWithDefault("param2", c._param2)
	c._status = "Configured"

	c._logger.Info(ctx, "MyComponentB has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references *crefer.References) {
	_another_component, err := references.GetOneRequired(crefer.NewDescriptor("myservice", "mycomponent-b", "*", "*", "1.0"))

	if err != nil {
		panic("Error: Another Component is not in refs")
	}

	c._another_component = _another_component.(MyComponentB)
	c._status = "Configured"

	c._logger.Info(ctx, "MyComponentA's references have been defined.")
}

func (c *MyComponentA) isOpen() bool {
	return c._opened
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) {
	c._opened = true
	c._status = "Open"
	c._logger.Info(ctx, "MyComponentA has been opened.")
	// artificial problem
	c.MyTask(ctx)
}

func (c *MyComponentA) Close(ctx context.Context) {
	c._opened = false
	c._status = "Closed"
	c._logger.Info(ctx, "MyComponentA has been closed.")
}

func (c *MyComponentA) MyTask(ctx context.Context) {
	// create an artificial problem
	c._logger.Error(ctx, errors.New("Logging demo. Error created"), "Error created.")
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentA) UnsetReferences(ctx context.Context) {
	c._another_component = nil
	c._status = "Un-referenced"
	c._logger.Info(ctx, "References cleared")
}

// Clears component state.
// - correlationId: (optional) transaction id to trace execution through call chain.
func (c *MyComponentA) Clear(ctx context.Context) {
	c.dummy_variable = nil
	c._status = ""
	c._logger.Info(ctx, "Resources cleared")
}
```

```
import (
      cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)

MyFactory1 := cbuild.NewFactory()

MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponentA", "default", "*", "1.0"), NewMyComponentA)
MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponent-b", "default", "*", "1.0"), NewMyComponentB)
```

```
import (
      cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
)

// Creating a process container
type MyProcess struct {
	*cproc.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := MyProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("myservice", "My service running as a process")
	c.SetConfigPath("./config/config.yml")
	MyFactory1 := cbuild.NewFactory()

	MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponentA", "default", "*", "1.0"), NewMyComponentA)
	MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponent-b", "default", "*", "1.0"), NewMyComponentB)

	c.AddFactory(MyFactory1)
	return &c
}
```

```
---
- descriptor: "myservice:mycomponentA:*:default:1.0"
  param1: XYZ
  param2: 987

- descriptor: myservice:mycomponent-b:*:*:1.0
  param1: XYZ
  param2: 987

```

```

func main() {
	proc := NewMyProcess()
	proc.Run(context.Background(), os.Environ())
}
```

```
import (
	"context"
	"errors"
	"os"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	crun "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

type MyComponentB struct {
	crefer.IReferences
	crefer.IUnreferenceable
	cconfig.IConfigurable
	crun.IOpenable
	crun.ICleanable

	_status string
	_param1 string
	_param2 int
	_opened bool
	_logger clog.ILogger

	dummy_variable interface{}
}

// Creates a new instance of the component.
func NewMyComponentB() *MyComponentB {
	component := &MyComponentB{
		_status: "Created",
		_param1: "ABC2",
		_param2: 456,
		_opened: false,
		_logger: clog.NewConsoleLogger(),

		dummy_variable: "resource variable",
	}

	component._logger.SetLevel(5)
	component._logger.Info(context.Background(), "MyComponentB has been created.")

	return component
}

func (c *MyComponentB) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._param1 = config.GetAsStringWithDefault("param1", c._param1)
	c._param2 = config.GetAsIntegerWithDefault("param2", c._param2)

	c._logger.Info(ctx, "MyComponentB has been configured.")
}

func (c *MyComponentB) SetReferences(ctx context.Context, references *crefer.References) {
	// pass
}

func (c *MyComponentB) isOpen() bool {
	// pass
	return true
}

func (c *MyComponentB) Open(ctx context.Context) {
	// pass
}

func (c *MyComponentB) Close(ctx context.Context) {
	// pass
}

func (c *MyComponentB) MyTask(ctx context.Context) {
	// pass
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentB) UnsetReferences(ctx context.Context) {
	// pass
}

// Clears component state.
// - correlationId: (optional) transaction id to trace execution through call chain.
func (c *MyComponentB) Clear(ctx context.Context) {
	// pass
}

type MyComponentA struct {
	crefer.IReferences
	crefer.IUnreferenceable
	cconfig.IConfigurable
	crun.IOpenable
	crun.ICleanable

	_logger clog.ILogger

	_status string
	_param1 string
	_param2 int
	_opened bool

	dummy_variable interface{}

	_another_component interface{}
}

// Creates a new instance of the component.
func NewMyComponentA() *MyComponentA {
	component := &MyComponentA{
		_status: "Created",
		_param1: "ABC2",
		_param2: 456,
		_opened: false,
		_logger: clog.NewConsoleLogger(),

		dummy_variable: "dummy_variable setted",
	}

	component._logger.SetLevel(5)
	component._logger.Info(context.Background(), "MyComponentA has been created.")

	return component
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._param1 = config.GetAsStringWithDefault("param1", c._param1)
	c._param2 = config.GetAsIntegerWithDefault("param2", c._param2)
	c._status = "Configured"

	c._logger.Info(ctx, "MyComponentB has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references *crefer.References) {
	_another_component, err := references.GetOneRequired(crefer.NewDescriptor("myservice", "mycomponent-b", "*", "*", "1.0"))

	if err != nil {
		panic("Error: Another Component is not in refs")
	}

	c._another_component = _another_component.(MyComponentB)
	c._status = "Configured"

	c._logger.Info(ctx, "MyComponentA's references have been defined.")
}

func (c *MyComponentA) isOpen() bool {
	return c._opened
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) {
	c._opened = true
	c._status = "Open"
	c._logger.Info(ctx, "MyComponentA has been opened.")
	// artificial problem
	c.MyTask(ctx)
}

func (c *MyComponentA) Close(ctx context.Context) {
	c._opened = false
	c._status = "Closed"
	c._logger.Info(ctx, "MyComponentA has been closed.")
}

func (c *MyComponentA) MyTask(ctx context.Context) {
	// create an artificial problem
	c._logger.Error(ctx, errors.New("Logging demo. Error created"), "Error created.")
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentA) UnsetReferences(ctx context.Context) {
	c._another_component = nil
	c._status = "Un-referenced"
	c._logger.Info(ctx, "References cleared")
}

// Clears component state.
// - correlationId: (optional) transaction id to trace execution through call chain.
func (c *MyComponentA) Clear(ctx context.Context) {
	c.dummy_variable = nil
	c._status = ""
	c._logger.Info(ctx, "Resources cleared")
}

// Creating a process container
type MyProcess struct {
	*cproc.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := MyProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("myservice", "My service running as a process")
	c.SetConfigPath("./config/config.yml")
	MyFactory1 := cbuild.NewFactory()

	MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponentA", "default", "*", "1.0"), NewMyComponentA)
	MyFactory1.RegisterType(crefer.NewDescriptor("myservice", "mycomponent-b", "default", "*", "1.0"), NewMyComponentB)

	c.AddFactory(MyFactory1)
	return &c
}

func main() {
	proc := NewMyProcess()
	proc.Run(context.Background(), os.Environ())
}

```

```
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	logdatadog "github.com/pip-services4/pip-services4-go/pip-services4-datadog-go/log"
)

func main() {

	logger := logdatadog.NewDataDogLogger()

	logger.Configure(context.Background(), cconfig.NewConfigParamsFromTuples(
		"credential.access_key", "827349874395872349875493",
	))

	logger.SetLevel(5)
	_ = logger.Open(context.Background())
	logger.Info(context.Background(), "My message")

}

```

```
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	logelastic "github.com/pip-services4/pip-services4-go/pip-services4-elasticsearch-go/log"
)

func main() {

	logger := logelastic.NewElasticSearchLogger()
	logger.Configure(context.Background(), cconfig.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 9200,
	))

	logger.SetLevel(5)
	_ = logger.Open(context.Background())
	logger.Info(context.Background(), "My message")

}
```

```
import (
	"context"

	cconfig "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	logaws "github.com/pip-services4/pip-services4-go/pip-services4-aws-go/log"
)

func main() {

	logger := logaws.NewCloudWatchLogger()
	logger.Configure(context.Background(), cconfig.NewConfigParamsFromTuples(
		"stream", "mystream",
		"group", "mygroup",
		"connection.region", "us-east-1",
		"connection.access_id", "XXXXXXXXXXX",
		"connection.access_key", "XXXXXXXXXXX",
	))

	logger.SetLevel(5)
	_ = logger.Open(context.Background())
	logger.Info(context.Background(), "My message")
}
```

```

```

```
// ElasticSearchLogger sends a log to ElasticSearch
elasticSearchLogger := elasticlog.NewElasticSearchLogger()
// Console logger writes messages to console
consoleLogger := clog.NewConsoleLogger()

```

```
config := cconf.NewConfigParamsFromTuples(
	// Common config
	"source", "my_component_log",
	"level", clog.LevelDebug,

	// Elastic config
	"index", "log",
	"daily", true,
	"date_format", "yyyyMMdd",
	"connection.host", "localhost",
	"connection.port", 9200,
	"connection.protocol", "http",
)
```

```
elasticSearchLogger.Configure(context.Background(), config)
_ = elasticSearchLogger.Open(context.Background(), "123")
```

```
consoleLogger.Configure(context.Background(), config)
```

```
references := cref.NewReferencesFromTuples(context.Background(),
	cref.NewDescriptor("my-component", "logger", "console", "default", "1.0"), elasticSearchLogger,
	cref.NewDescriptor("my-component", "logger", "elasticsearch", "default", "1.0"), consoleLogger,
)

```

```
logger := clog.NewCompositeLogger()
```

```
logger.SetReferences(context.Background(), references)
```

```
logger.Info(context.Background(), "Composite logger is configured and ready for using")
logger.Warn(context.Background(), "Example warning")
logger.Error(context.Background(), errors.New("Example error"), "Error message")
logger.Debug(context.Background(), "Debug info")
logger.Fatal(context.Background(), errors.New("Fatal error"), "Error that crash the process")
```

```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "log123-20220126",
        "_type" : "log_message",
        "_id" : "bf18e9659fb043668f8c006d4e9aba26",
        "_score" : 1.0,
        "_source" : {
          "time" : "2022-01-26T23:28:10.0825714Z",
          "source" : "my_component_log",
          "level" : 4,
          "correlation_id" : "123",
          "error" : {
            "type" : "",
            "category" : "",
            "status" : 0,
            "code" : "",
            "message" : "",
            "details" : null,
            "correlation_id" : "",
            "cause" : "",
            "stack_trace" : ""
          },
          "message" : "Composite logger is configured and ready for using"
        }
      },
      {
        "_index" : "log123-20220126",
        "_type" : "log_message",
        "_id" : "25cc4c2c81984a348f640e7435891e2b",
        "_score" : 1.0,
        "_source" : {
          "time" : "2022-01-26T23:28:10.0832355Z",
          "source" : "my_component_log",
          "level" : 3,
          "correlation_id" : "123",
          "error" : {
            "type" : "",
            "category" : "",
            "status" : 0,
            "code" : "",
            "message" : "",
            "details" : null,
            "correlation_id" : "",
            "cause" : "",
            "stack_trace" : ""
          },
          "message" : "Example warning"
        }
      },
      {
        "_index" : "log123-20220126",
        "_type" : "log_message",
        "_id" : "7e671ce009be46078272eae6dcab0328",
        "_score" : 1.0,
        "_source" : {
          "time" : "2022-01-26T23:28:10.0832355Z",
          "source" : "my_component_log",
          "level" : 2,
          "correlation_id" : "123",
          "error" : {
            "type" : "",
            "category" : "Unknown",
            "status" : 500,
            "code" : "UNKNOWN",
            "message" : "Example error",
            "details" : null,
            "correlation_id" : "",
            "cause" : "",
            "stack_trace" : ""
          },
          "message" : "Error message"
        }
      },
      {
        "_index" : "log123-20220126",
        "_type" : "log_message",
        "_id" : "f9879db7638947ab8ae43c475c6d8e78",
        "_score" : 1.0,
        "_source" : {
          "time" : "2022-01-26T23:28:10.0846687Z",
          "source" : "my_component_log",
          "level" : 5,
          "correlation_id" : "123",
          "error" : {
            "type" : "",
            "category" : "",
            "status" : 0,
            "code" : "",
            "message" : "",
            "details" : null,
            "correlation_id" : "",
            "cause" : "",
            "stack_trace" : ""
          },
          "message" : "Debug info"
        }
      },
      {
        "_index" : "log123-20220126",
        "_type" : "log_message",
        "_id" : "4261be380da94e18923f726d60a91bc7",
        "_score" : 1.0,
        "_source" : {
          "time" : "2022-01-26T23:28:10.0846687Z",
          "source" : "my_component_log",
          "level" : 1,
          "correlation_id" : "123",
          "error" : {
            "type" : "",
            "category" : "Unknown",
            "status" : 500,
            "code" : "UNKNOWN",
            "message" : "Fatal error",
            "details" : null,
            "correlation_id" : "",
            "cause" : "",
            "stack_trace" : ""
          },
          "message" : "Error that crash the process"
        }
      }
    ]
  }
}

```
