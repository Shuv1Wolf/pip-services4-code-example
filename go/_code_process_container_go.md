```
import ccont "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
```

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type MyComponentA struct {
	opened bool
}

func NewMyComponentA() *MyComponentA {
	defer fmt.Println("MyComponentA has been created.")
	c := &MyComponentA{}
	return c
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	fmt.Println("MyComponentA has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references crefer.IReferences) {
	fmt.Println("MyComponentA's references have been defined.")
}

func (c *MyComponentA) UnsetReferences() {
	fmt.Println("References cleared")
}

func (c *MyComponentA) IsOpen() bool {
	return c.opened
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) error {
	fmt.Println("MyComponentA has been opened.")
	return nil
}

func (c *MyComponentA) Close(ctx context.Context, correlationId string) error {
	fmt.Println("MyComponentA has been closed.")
	return nil
}

func (c *MyComponentA) Execute(ctx context.Context, correlationId string, args cexec.Parameters) (result any, err error) {
	fmt.Println("Executing")
	result = args
	return result, nil
}
```

```
import cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(crefer.NewDescriptor("myservice", "MyComponentA", "*", "*", "1.0"), NewMyComponentA)
	return c
}
```

```
---
# Context information
- descriptor: "pip-services:context-info:default:default:1.0"
  name: myservice
  description: My service running in a process container

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: {{LOG_LEVEL}}{{^LOG_LEVEL}}info{{/LOG_LEVEL}}

# Performance counters that posts values to log
- descriptor: "pip-services:counters:log:default:1.0"
  
# My component
- descriptor: "myservice:MyComponentA:default:*:1.0"
```

```
import (
        crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ccont "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
)

type MyProcess struct {
	*ccont.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := &MyProcess{}
	c.ProcessContainer = ccont.NewProcessContainer("myservice", "My service running as a process")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}
```

```
func main() {
	proc := NewMyProcess()
    // proc.SetConfigPath("./config/config.yml")
	proc.Run(context.Background(), os.Args)
}
```

```
import (
	"context"
	"fmt"
	"os"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ccont "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
)

func main() {
	proc := NewMyProcess()
	// proc.SetConfigPath("./config/config.yml")
	proc.Run(context.Background(), os.Args)
}

type MyProcess struct {
	*ccont.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := &MyProcess{}
	c.ProcessContainer = ccont.NewProcessContainer("myservice", "My service running as a process")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(crefer.NewDescriptor("myservice", "MyComponentA", "*", "*", "1.0"), NewMyComponentA)
	return c
}

type MyComponentA struct {
	opened bool
}

func NewMyComponentA() *MyComponentA {
	defer fmt.Println("MyComponentA has been created.")
	c := &MyComponentA{}
	return c
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	fmt.Println("MyComponentA has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references crefer.IReferences) {
	fmt.Println("MyComponentA's references have been defined.")
}

func (c *MyComponentA) UnsetReferences() {
	fmt.Println("References cleared")
}

func (c *MyComponentA) IsOpen() bool {
	return c.opened
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) error {
	fmt.Println("MyComponentA has been opened.")
	return nil
}

func (c *MyComponentA) Close(ctx context.Context, correlationId string) error {
	fmt.Println("MyComponentA has been closed.")
	return nil
}

func (c *MyComponentA) Execute(ctx context.Context, correlationId string, args cexec.Parameters) (result any, err error) {
	fmt.Println("Executing")
	result = args
	return result, nil
}
```
