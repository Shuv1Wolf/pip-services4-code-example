```
type IReferences interface {

	Put(ctx context.Context, locator any, component any)

	Remove(ctx context.Context, locator any) any

	RemoveAll(ctx context.Context, locator any) []any

	GetAllLocators() []any

	GetAll() []any

	GetOptional(locator any) []any

	GetRequired(locator any) ([]any, error)

	GetOneOptional(locator any) any

	GetOneRequired(locator any) (any, error)

	Find(locator any, required bool) ([]any, error)
}
```

```
type IUnreferenceable interface {
	UnsetReferences(ctx context.Context)
}
```

```
import (
	"context"
	"fmt"

	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type Worker interface {
	Do(level int, message string)
}

type Worker1 struct {
	_defaultName string
}

func NewWorker1(name string) *Worker1 {
	if name != "" {
		return &Worker1{_defaultName: "default name1"}
	}
	return &Worker1{_defaultName: "default name1"}

}

func (c *Worker1) Do(level int, message string) {
	fmt.Printf("Write to %v.%v message: %v", c._defaultName, level, message)
}

type Worker2 struct {
	_defaultName string
}

func NewWorker2(name string) *Worker2 {
	if name != "" {
		return &Worker2{_defaultName: "default name2"}
	}
	return &Worker2{_defaultName: "default name2"}

}

func (c *Worker2) Do(level int, message string) {
	fmt.Printf("Write to %v.%v message: %v", c._defaultName, level, message)
}

type SimpleController struct {
	_worker interface{}
}

func (c *SimpleController) SetReferences(ctx context.Context, references crefer.IReferences) {
	c._worker, _ = references.GetOneRequired(111)
}

func (c *SimpleController) UnsetReferences(ctx context.Context) {
	c._worker = nil
}

func (c *SimpleController) Greeting(ctx context.Context, name string) {

	c._worker.(Worker).Do(5, "Hello, "+name+"!")
}
```

```
references := crefer.NewReferencesFromTuples(context.Background(),
		111, mymodule.NewWorker1("worker1"),
		222, mymodule.NewWorker2("worker2"),
	)

	controller := mymodule.NewSimpleController()
	controller.SetReferences(context.Background(), references)
	controller.Greeting(context.Background(), "world")
	controller.UnsetReferences(context.Background())
	controller = nil
```

```

type Descriptor struct {
	group   string
	typ     string
	kind    string
	name    string
	version string
}

func NewDescriptor(group string, typ string, kind string, name string, version string) *Descriptor {
	if "*" == group {
		group = ""
	}
	if "*" == typ {
		typ = ""
	}
	if "*" == kind {
		kind = ""
	}
	if "*" == name {
		name = ""
	}
	if "*" == version {
		version = ""
	}

	return &Descriptor{group: group, typ: typ, kind: kind, name: name, version: version}
}

func (c *Descriptor) Group() string {
	return c.group
}

func (c *Descriptor) Type() string {
	return c.typ
}

func (c *Descriptor) Kind() string {
	return c.kind
}

func (c *Descriptor) Name() string {
	return c.name
}

func (c *Descriptor) Version() string {
	return c.version
}

func matchField(field1 string, field2 string) bool {
	return field1 == "" || field2 == "" || field1 == field2
}

func (c *Descriptor) Match(descriptor *Descriptor) bool {
	return matchField(c.group, descriptor.Group()) &&
		matchField(c.typ, descriptor.Type()) &&
		matchField(c.kind, descriptor.Kind()) &&
		matchField(c.name, descriptor.Name()) &&
		matchField(c.version, descriptor.Version())
}

func exactMatchField(field1 string, field2 string) bool {
	if field1 == "" && field2 == "" {
		return true
	}
	if field1 == "" || field2 == "" {
		return false
	}
	return field1 == field2
}

func (c *Descriptor) ExactMatch(descriptor *Descriptor) bool {
	return exactMatchField(c.group, descriptor.Group()) &&
		exactMatchField(c.typ, descriptor.Type()) &&
		exactMatchField(c.kind, descriptor.Kind()) &&
		exactMatchField(c.name, descriptor.Name()) &&
		exactMatchField(c.version, descriptor.Version())
}

func (c *Descriptor) IsComplete() bool {
	return c.group != "" && c.typ != "" && c.kind != "" &&
		c.name != "" && c.version != ""
}

func (c *Descriptor) Equals(value interface{}) bool {
	descriptor, ok := value.(*Descriptor)
	if ok {
		return c.Match(descriptor)
	}
	return false
}

func (c *Descriptor) String() string {
	result := ""
	if c.group == "" {
		result += "*"
	} else {
		result += c.group
	}
	if c.typ == "" {
		result += ":*"
	} else {
		result += ":" + c.typ
	}
	if c.kind == "" {
		result += ":*"
	} else {
		result += ":" + c.kind
	}
	if c.name == "" {
		result += ":*"
	} else {
		result += ":" + c.name
	}
	if c.version == "" {
		result += ":*"
	} else {
		result += ":" + c.version
	}
	return result
}

func ParseDescriptorFromString(value string) (*Descriptor, error) {
	if value == "" {
		return nil, nil
	}

	tokens := strings.Split(value, ":")
	if len(tokens) != 5 {
		return nil, errors.NewConfigError("", "BAD_DESCRIPTOR", "Descriptor "+value+" is in wrong format")
	}

	return NewDescriptor(strings.TrimSpace(tokens[0]), strings.TrimSpace(tokens[1]),
		strings.TrimSpace(tokens[2]), strings.TrimSpace(tokens[3]), strings.TrimSpace(tokens[4])), nil
}
```

```
pip-services:logger:console:logger1:1.0
pip-services:logger:elasticsearch:logger2:1.0
my-library:logger:mylog:logger3:1.0
```

```
*:logger:*:*:1.0
```

```
*:logger:console:*:1.0
```

```
*:logger:*:logger2:1.0
```

```

my_library:*:*:*:*
```

```
///

func (c *SimpleController) SetReferences(ctx context.Context, references crefer.IReferences) {
	c._worker, _ = references.GetOneRequired(crefer.NewDescriptor("*", "worker", "worker1", "*", "1.0"))
}

///

references := crefer.NewReferencesFromTuples(context.Background(),
	crefer.NewDescriptor("sample", "worker", "worker1", "111", "1.0"), mymodule.NewWorker1(""),
	crefer.NewDescriptor("sample", "worker", "worker2", "222", "1.0"), mymodule.NewWorker2(""),
)

controller := mymodule.NewSimpleController()
controller.SetReferences(context.Background(), references)
controller.Greeting(context.Background(), "world")
controller.UnsetReferences(context.Background())
controller = nil
```

```
type DependencyResolver struct {
	dependencies map[string]interface{}
	references   IReferences
}

func NewDependencyResolver() *DependencyResolver {
	return &DependencyResolver{
		dependencies: map[string]interface{}{},
		references:   nil,
	}
}

func NewDependencyResolverWithParams(config *conf.ConfigParams, references IReferences) *DependencyResolver {
	c := NewDependencyResolver()

	if config != nil {
		c.Configure(context.Background(), config)
	}

	if references != nil {
		c.SetReferences(context.Background(), references)
	}

	return c
}

func (c *DependencyResolver) Configure(ctx context.Context, config *conf.ConfigParams) {
	dependencies := config.GetSection("dependencies")
	names := dependencies.Keys()
	for _, name := range names {
		locator := dependencies.Get(name)
		if locator == "" {
			continue
		}

		descriptor, err := ParseDescriptorFromString(locator)
		if err == nil {
			c.dependencies[name] = descriptor
		} else {
			c.dependencies[name] = locator
		}
	}
}

func (c *DependencyResolver) SetReferences(ctx context.Context, references IReferences) {
	c.references = references
}

func (c *DependencyResolver) Put(ctx context.Context, name string, locator interface{}) {
	c.dependencies[name] = locator
}

func (c *DependencyResolver) Locate(name string) interface{} {
	if name == "" {
		panic("Dependency name cannot be empty")
	}

	if c.references == nil {
		panic("References shall be set")
	}

	return c.dependencies[name]
}

func (c *DependencyResolver) GetOptional(name string) []interface{} {
	locator := c.Locate(name)
	if locator == nil {
		return []interface{}{}
	}
	return c.references.GetOptional(locator)
}

func (c *DependencyResolver) GetRequired(name string) ([]interface{}, error) {
	locator := c.Locate(name)
	if locator == nil {
		err := NewReferenceError("", name)
		return []interface{}{}, err
	}

	return c.references.GetRequired(locator)
}

func (c *DependencyResolver) GetOneOptional(name string) interface{} {
	locator := c.Locate(name)
	if locator == nil {
		return nil
	}
	return c.references.GetOneOptional(locator)
}

func (c *DependencyResolver) GetOneRequired(name string) (interface{}, error) {
	locator := c.Locate(name)
	if locator == nil {
		err := NewReferenceError("", name)
		return nil, err
	}
	return c.references.GetOneRequired(locator)
}

func (c *DependencyResolver) Find(name string, required bool) ([]interface{}, error) {
	if name == "" {
		panic("Name cannot be empty")
	}

	locator := c.Locate(name)
	if locator == nil {
		if required {
			err := NewReferenceError("", name)
			return []interface{}{}, err
		}
		return []interface{}{}, nil
	}

	return c.references.Find(locator, required)
}

func NewDependencyResolverFromTuples(tuples ...interface{}) *DependencyResolver {
	result := NewDependencyResolver()
	if len(tuples) == 0 {
		return result
	}

	for index := 0; index < len(tuples); index += 2 {
		if index+1 >= len(tuples) {
			break
		}

		name := convert.StringConverter.ToString(tuples[index])
		locator := tuples[index+1]

		result.Put(name, locator)
	}

	return result
}
```

```
type SimpleController struct {
	_worker            interface{}
	_depedencyResolver crefer.DependencyResolver
}

func NewSimpleController() *SimpleController {
	return &SimpleController{_depedencyResolver: *crefer.NewDependencyResolverFromTuples(
		"worker", crefer.NewDescriptor("*", "worker", "*", "*", "1.0"),
	)}
}

func (c *SimpleController) Configure(ctx context.Context, config *cconfig.ConfigParams) {
	c._depedencyResolver.Configure(ctx, config)
}

func (c *SimpleController) SetReferences(ctx, references crefer.IReferences) {
	c._depedencyResolver.SetReferences(ctx, references)
	c._worker, _ = c._depedencyResolver.GetOneRequired("worker")
}

func (c *SimpleController) UnsetReferences(ctx) {
	c._depedencyResolver = *crefer.NewDependencyResolver()
}

...

references := crefer.NewReferencesFromTuples(context.Background(),
	crefer.NewDescriptor("sample", "worker", "worker1", "111", "1.0"), mymodule.NewWorker1(""),
	crefer.NewDescriptor("sample", "worker", "worker2", "222", "1.0"), mymodule.NewWorker2(""),
)

config := cconfig.NewConfigParamsFromTuples(
	"dependencies.worker", "*:worker:worker1:111:1.0",
)

controller := mymodule.NewSimpleController()
controller.Configure(context.Background(),config)
controller.SetReferences(context.Background(),references)
controller.Greeting(context.Background(),"world")
controller.UnsetReferences(context.Background())
controller = nil
```

```
- descriptor: "sample-references:worker:worker1:*:1.0"
  default_name: "Worker1"

- descriptor: "sample-references:worker:worker2:*:1.0"
  default_name: "Worker2"

- descriptor: "sample-references:controller:default:default:1.0"
  default_name: "Sample"
  dependencies:
    workers: "sample-references:worker:worker2:*:1.0"
```

```
type Reference struct {
	locator   interface{}
	component interface{}
}

func NewReference(locator interface{}, component interface{}) *Reference {
	...
}

func (c *Reference) Component() interface{} {
	...
}

func (c *Reference) Locator() interface{} {
	...
}

func (c *Reference) Match(locator interface{}) bool {
	...
}
```
