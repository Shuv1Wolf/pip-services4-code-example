#### Step 1

```
type MyComponentA struct {
	param1 string
	param2 int
	open   bool

	status string
}

func NewMyComponentA() *MyComponentA {
	fmt.Println("MyComponentA has been created.")

	return &MyComponentA{
		param1: "ABC",
		param2: 123,
		open:   false,
		status: "Created",
	}
}
```

#### Configuration

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type MyComponentA struct {
	cconf.IConfigurable

	param1 string
	param2 int
	open   bool

	status string
}

// Creates a new instance of the component.
func NewMyComponentA() *MyComponentA {
	fmt.Println("MyComponentA has been created.")

	return &MyComponentA{
		param1: "ABC",
		param2: 123,
		open:   false,
		status: "Created",
	}
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.param1 = config.GetAsStringWithDefault("param1", "ABC")
	c.param2 = config.GetAsIntegerWithDefault("param2", 123)
	c.status = "Configured"
	fmt.Println("MyComponentA has been configured.")
}
```

#### Referencing

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type MyComponentB struct {
	param1 string
	param2 int
	open   bool

	status string
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}

	status string
}

// Creates a new instance of the component.
func NewMyComponentB() *MyComponentB {
	fmt.Println("MyComponentB has been created.")

	return &MyComponentB{
		param1: "ABC2",
		param2: 456,
		open:   false,
		status: "Created",
	}
}

// Creates a new instance of the component.
func NewMyComponentA() *MyComponentA {
	fmt.Println("MyComponentA has been created.")

	return &MyComponentA{
		param1: "ABC",
		param2: 123,
		open:   false,
		status: "Created",
	}
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.param1 = config.GetAsStringWithDefault("param1", "ABC")
	c.param2 = config.GetAsIntegerWithDefault("param2", 123)
	c.status = "Configured"
	fmt.Println("MyComponentA has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references refer.IReferences) {
	component, err := references.GetOneRequired(
		refer.NewDescriptor("myservice", "mycomponent-b", "*", "*", "1.0"),
	)
	if err == nil {
		c.anotherComponent = component.(*MyComponentB)
		c.status = "Configured"
		fmt.Println("MyComponentA's references have been defined.")
	}

}
```

#### Opening

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
	// ...
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}

	status string
}

func (c *MyComponentA) IsOpen() bool {
	return c.open
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) error {
	c.open = true
	c.status = "Open"
	fmt.Println("MyComponentA has been opened.")
	return nil
}

```

#### Execution

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
	// ...
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}
	dummyVariable    string

	status string
}

// ...

func (c *MyComponentA) MyTask(ctx context.Context, correlationId string) {
	fmt.Println("Doing my business task")
	c.dummyVariable = "dummy value"
}
```

#### Closing

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
	// ...
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}
	dummyVariable    string

	status string
}

// ...

func (c *MyComponentA) Close(ctx context.Context, correlationId string) error {
	c.open = false
	c.status = "Closed"
	fmt.Println("MyComponentA has been closed.")

	return nil
}
```

#### Un-referencing

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
	// ...
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	refer.IUnreferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}
	dummyVariable    string

	status string
}

// ...

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentA) UnsetReferences(ctx context.Context) {
	c.anotherComponent = nil
	c.status = "Un-referenced"
	fmt.Println("References cleared")

}
```

#### Destruction

```
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
    // ...
}
  
type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	refer.IUnreferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}
	dummyVariable    string

	status string
}

// ...

component := NewMyComponentA()

defer func() {
	if !component.IsOpen() {
		fmt.Println("Component destroyed")
	}
}()

// ...

component.Close(context.Background(), nil)
```

#### Final code

````
import (
	"context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	run "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
)

type MyComponentB struct {
	param1 string
	param2 int
	open   bool

	status string
}

// Creates a new instance of the component.
func NewMyComponentB() *MyComponentB {
	fmt.Println("MyComponentB has been created.")

	return &MyComponentB{
		param1: "ABC2",
		param2: 456,
		open:   false,
		status: "Created",
	}
}

func (c *MyComponentB) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// pass
}

func (c *MyComponentB) SetReferences(ctx context.Context, references refer.IReferences) {
	// pass
}

func (c *MyComponentB) IsOpen() bool {
	// pass
	return true
}

func (c *MyComponentB) Open(ctx context.Context, correlationId string) error {
	// pass
	return nil
}

func (c *MyComponentB) Close(ctx context.Context, correlationId string) error {
	// pass
	return nil
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentB) UnsetReferences(ctx context.Context) {
	// pass
}

func (c *MyComponentB) MyTask(ctx context.Context, correlationId string) {
	// pass
}

type MyComponentA struct {
	cconf.IConfigurable
	refer.IReferenceable
	refer.IUnreferenceable
	run.IOpenable

	param1 string
	param2 int
	open   bool

	anotherComponent interface{}
	dummyVariable    string

	status string
}

// Creates a new instance of the component.
func NewMyComponentA() *MyComponentA {
	fmt.Println("MyComponentA has been created.")

	return &MyComponentA{
		param1: "ABC",
		param2: 123,
		open:   false,
		status: "Created",
	}
}

func (c *MyComponentA) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.param1 = config.GetAsStringWithDefault("param1", "ABC")
	c.param2 = config.GetAsIntegerWithDefault("param2", 123)
	c.status = "Configured"
	fmt.Println("MyComponentA has been configured.")
}

func (c *MyComponentA) SetReferences(ctx context.Context, references refer.IReferences) {
	component, err := references.GetOneRequired(
		refer.NewDescriptor("myservice", "mycomponent-b", "*", "*", "1.0"),
	)
	if err == nil {
		c.anotherComponent = component.(*MyComponentB)
		c.status = "Configured"
		fmt.Println("MyComponentA's references have been defined.")
	}

}

func (c *MyComponentA) IsOpen() bool {
	return c.open
}

func (c *MyComponentA) Open(ctx context.Context, correlationId string) error {
	c.open = true
	c.status = "Open"
	fmt.Println("MyComponentA has been opened.")

	return nil
}

func (c *MyComponentA) Close(ctx context.Context, correlationId string) error {
	c.open = false
	c.status = "Closed"
	fmt.Println("MyComponentA has been closed.")

	return nil
}

// Unsets (clears) previously set references to dependent components.
func (c *MyComponentA) UnsetReferences(ctx context.Context) {
	c.anotherComponent = nil
	c.status = "Un-referenced"
	fmt.Println("References cleared")

}

func (c *MyComponentA) MyTask(ctx context.Context, correlationId string) {
	fmt.Println("Doing my business task")
	c.dummyVariable = "dummy value"
}
```
````

### Executing our code

```
import (
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	// Step 1 - Creation
	myComponentA := NewMyComponentA()
	myComponentB := NewMyComponentB()

	// Step 2 - Configure the component
	myComponentA.Configure(ctx context.Context, cconf.NewConfigParamsFromTuples(
		"param1", "XYZ",
		"param2", "987",
	))

	// Step 3 - Referencing
	// Set references to the component
	myComponentA.SetReferences(ctx context.Context, refer.NewReferencesFromTuples(
		refer.NewDescriptor("myservice", "mycomponent-b", "default", "default", "1.0"), myComponentB,
	))

	// Step 4 - Openning
	myComponentA.Open(ctx context.Context, "123")

	// Step 5 - Execution
	myComponentA.MyTask(ctx context.Context, "123")

	// Step 6 - Closing
	myComponentA.Close(ctx context.Context, "123")

	// Step 7 - Un-referencing
	myComponentA.UnsetReferences(ctx context.Context)

	// Step 8 - Destruction
	defer func() {
		if !NewMyComponentA().IsOpen() {
			fmt.Println("Component destroyed")
		}
	}()
}

```

### Assembling a service from our component

```
import (
	build "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

// Creating a factory

type MyClassFactory struct {
	*build.Factory
}

func NewMyClassFactory() *MyClassFactory {
	c := MyClassFactory{}
	c.Factory = build.NewFactory()

	ComponentADescriptor := refer.NewDescriptor("myservice", "mycomponentA", "default", "*", "1.0")
	ComponentBDescriptor := refer.NewDescriptor("myservice", "mycomponent-b", "default", "*", "1.0")

	c.RegisterType(ComponentADescriptor, NewMyComponentA)
	c.RegisterType(ComponentBDescriptor, NewMyComponentB)

	return &c
}
```

```
import (
	container "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
)

type MyProcess struct {
	*container.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := &MyProcess{
		ProcessContainer: container.NewProcessContainer("myservice", "My service running as a process"),
	}
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyClassFactory())

	return c
}
```

```
func main() {
	runner := NewMyProcess()
	runner.Run(os.Environ())
}
```
