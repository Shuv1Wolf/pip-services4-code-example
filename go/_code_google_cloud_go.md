```
// Service's descriptor is defined in the configuration file
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service *MyService
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunction(c)
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)

	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service= res.(*MyService)
}

func (c *MyCloudFunction) action(res http.ResponseWriter, req *http.Request) {
	var body map[string]string

	_ = gcputil.CloudFunctionRequestHelper.DecodeBody(req, &body)
	defer req.Body.Close()

	name := body["name"]

	result, err := c.service.Greetings(req.Context(), name)
	httpcontrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Register() {
	c.RegisterAction("greetings", nil, c.action)
}
```

```
# Service
- descriptor: "mygroup:service:default:service:1.0"

```

```
// Service is added as a dependency
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service *MyService
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunction(c)
	c.SetConfigPath("./config.yaml")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)

	res, err := c.DependencyResolver.GetOneRequired("service")
	if err != nil {
		panic("Controller is not found")
	}

	c.service= res.(*MyService)
}

func (c *MyCloudFunction) action(res http.ResponseWriter, req *http.Request) {
	var body map[string]string

	_ = gcputil.CloudFunctionRequestHelper.DecodeBody(req, &body)
	defer req.Body.Close()

	name := body["name"]

	result, err := c.service.Greetings(req.Context(), name)
	httpcontrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Register() {
	c.RegisterAction("greetings", nil, c.action)
}

```

```
type MyService struct{}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}
```

```
type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "default", "controller", "1.0"), NewMyService)
	return c
}
```

```
import (
	"context"
	"net/http"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	gcputil "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/utils"
	httpcontrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyService struct{}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "default", "controller", "1.0"), NewMyService)
	return c
}

type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service *MyService
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunction(c)
	c.SetConfigPath("./config.yaml")
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "controller", "default", "controller", "*"))
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
        // Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// res, err := c.DependencyResolver.GetOneRequired("conroller")
	// Comment out the next line of code when using the dependency resolver, uncomment for configuration file
	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service = res.(*MyService)
}

func (c *MyCloudFunction) action(res http.ResponseWriter, req *http.Request) {
	var body map[string]string

	_ = gcputil.CloudFunctionRequestHelper.DecodeBody(req, &body)
	defer req.Body.Close()

	name := body["name"]

	result, err := c.service.Greetings(req.Context(), name)
	httpcontrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Register() {
	c.RegisterAction("greetings", nil, c.action)
}

```

```
// Controller's descriptor is defined in the configuration file
type MyCommandableCloudFunction struct {
	*gcpcont.CommandableCloudFunction
	service *MyService
}

func NewMyCommandableCloudFunction() *MyCommandableCloudFunction {
	c := &MyCommandableCloudFunction{}
	c.CommandableCloudFunction = gcpcont.NewCommandableCloudFunction()
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCommandableCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CommandableCloudFunction.SetReferences(ctx, references)
	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service = res.(*MyService)
}
```

```
type MyCommandableCloudFunction struct {
	*gcpcont.CommandableCloudFunction
	service *MyService
}

func NewMyCommandableCloudFunction() *MyCommandableCloudFunction {
	c := &MyCommandableCloudFunction{}
	c.CommandableCloudFunction = gcpcont.NewCommandableCloudFunction()
	c.SetConfigPath("./config.yaml")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCommandableCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CommandableCloudFunction.SetReferences(ctx, references)
	res, err := c.DependencyResolver.GetOneRequired("service")
	if err != nil {
		panic("Controller is not found")
	}

	c.service = res.(*MyService)
}
```

```
type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"), NewMyService)
	return c
}
```

```
type MyService struct {
	commandSet *MyCommandSet
}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) GetCommandSet() *MyCommandSet {
	if c.commandSet == nil {
		c.commandSet = NewMyCommandSet(c)
	}

	return c.commandSet
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}
```

```
type MyCommandSet struct {
	*ccomand.CommandSet
	service *MyService
}

func NewMyCommandSet(controller *MyService) *MyCommandSet {
	c := &MyCommandSet{}
	c.service = controller
	c.AddCommand(c.MakeGreeting())
	return c
}

func (c *MyCommandSet) MakeGreeting() *ccomand.Command {
	return ccomand.NewCommand(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (any, error) {
			name := args.GetAsString("name")
			return c.service.Greetings(ctx, name)
		},
	)
}
```

```
import (
	"context"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type MyCommandSet struct {
	*ccomand.CommandSet
	service *MyService
}

func NewMyCommandSet(controller *MyService) *MyCommandSet {
	c := &MyCommandSet{}
	c.service = controller
	c.AddCommand(c.MakeGreeting())
	return c
}

func (c *MyCommandSet) MakeGreeting() *ccomand.Command {
	return ccomand.NewCommand(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (any, error) {
			name := args.GetAsString("name")
			return c.service.Greetings(ctx, name)
		},
	)
}

type MyService struct {
	commandSet *MyCommandSet
}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) GetCommandSet() *MyCommandSet {
	if c.commandSet == nil {
		c.commandSet = NewMyCommandSet(c)
	}

	return c.commandSet
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"), NewMyService)
	return c
}

type MyCommandableCloudFunction struct {
	*gcpcont.CommandableCloudFunction
	service *MyService
}

func NewMyCommandableCloudFunction() *MyCommandableCloudFunction {
	c := &MyCommandableCloudFunction{}
	c.CommandableCloudFunction = gcpcont.NewCommandableCloudFunction()
	c.SetConfigPath("./config.yaml")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCommandableCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CommandableCloudFunction.SetReferences(ctx, references)
	res, err := c.DependencyResolver.GetOneRequired("service")
	if err != nil {
		panic("Controller is not found")
	}

	c.service = res.(*MyService)
}

```

```
// Controller's and service’s descriptors are defined in the configuration file
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service  *MyService
	controller     *MyCloudController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
	serv, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	contr, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "contorller", "gcp-function", "contorller", "*"))
	if err != nil {
		panic("Service is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*MyCloudContorller)
}
```

```
# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"  
# Service
- descriptor: "mygroup:service:default:service:1.0"
# Controller
- descriptor: "mygroup:controller:gcp-function:function:1.0"
```

```
// Controller and service are added as dependencies
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service    *MyService
	controller *MyCloudController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "controller", "gcp-function", "function", "*"))
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
	serv, err := c.DependencyResolver.GetOneRequired("service")
	if err != nil {
		panic("Controller is not found")
	}

	contr, err := c.DependencyResolver.GetOneRequired("controller")
	if err != nil {
		panic("Service is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*MyCloudContorller)
}
```

```
type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"), NewMyService)
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "gcp-function", "function", "1.0"), NewMyCloudController)
	return c
}
```

```
type MyCloudController struct {
	*gcpctrl.CloudFunctionController
	service *MyService
	headers map[string]string
}

func NewMyCloudController() *MyCloudController {
	c := &MyCloudController{}
	c.CloudFunctionController = gcpctrl.NewCloudFunctionController("")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	return c
}

func (c *MyCloudController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunctionController.SetReferences(ctx, references)
	res, err := c.DependencyResolver.GetOneRequired("service")
	if err != nil {
		panic("Service is not found")
	}

	c.service = res.(*MyService)
}

func (c *MyCloudController) Register() {
	c.RegisterAction(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("body", cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String)).Schema,
		func(res http.ResponseWriter, req *http.Request) {
			params := gcputil.CloudFunctionRequestHelper.GetParameters(req)
			name := params.GetAsStringWithDefault("name", "default name")

			result, err := c.service.Greetings(req.Context(), name)

			for key, value := range c.headers {
				res.Header().Add(key, value)
			}

			rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
		},
	)
}
```

```
type MyService struct{}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}
```

```
import (
	"context"
	"net/http"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	gcpctrl "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/controllers"
	gcputil "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/utils"
	rpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyCloudController struct {
	*gcpctrl.CloudFunctionController
	service *MyService
	headers map[string]string
}

func NewMyCloudController() *MyCloudController {
	c := &MyCloudController{}
	c.CloudFunctionController = gcpctrl.NewCloudFunctionController("")
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	return c
}

func (c *MyCloudController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunctionController.SetReferences(ctx, references)
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// res, err := c.DependencyResolver.GetOneRequired("service")
	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	if err != nil {
		panic("Service is not found")
	}

	c.service = res.(*MyService)
}

func (c *MyCloudController) Register() {
	c.RegisterAction(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("body", cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String)).Schema,
		func(res http.ResponseWriter, req *http.Request) {
			params := gcputil.CloudFunctionRequestHelper.GetParameters(req)
			name := params.GetAsStringWithDefault("name", "default name")

			result, err := c.service.Greetings(req.Context(), name)

			for key, value := range c.headers {
				res.Header().Add(key, value)
			}

			rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
		},
	)
}

type MyService struct{}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"), NewMyService)
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "gcp-function", "function", "1.0"), NewMyCloudController)
	return c
}

type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service    *MyService
	controller *MyCloudController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	// Comment out the next two lines of code when using a configuration file, uncomment to use the dependency resolver
	// c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	// c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "controller", "gcp-function", "function", "*"))
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// serv, err := c.DependencyResolver.GetOneRequired("service")
	// Comment out the next line of code when using the dependency resolver, uncomment for configuration file
	serv, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Service is not found")
	}

	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// contr, err := c.DependencyResolver.GetOneRequired("controller")
	// Comment out the next line of code when using the dependency resolver, uncomment for configuration file
	contr, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "controller", "gcp-function", "function", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*MyCloudController)
}
```

```
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service    *MyService
	controller *gcpctrl.CloudFunctionController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
	serv, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Service is not found")
	}

	contr, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "controller", "commandable-gcp-function", "function", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*gcpctrl.CloudFunctionController)
}
```

```
# Service
- descriptor: "mygroup:service:default:service:1.0"
# Controller
- descriptor: "mygroup:controller:commandable-gcp-function:function:1.0"
```

```
type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service    *MyService
	controller *gcpctrl.CloudFunctionController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "controller", "commandable-gcp-function", "function", "*"))
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
        serv, err := c.DependencyResolver.GetOneRequired("service")
        if err != nil {
		panic("Service is not found")
	}

        contr, err := c.DependencyResolver.GetOneRequired("controller")
        if err != nil {
		panic("Controller is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*gcpctrl.CloudFunctionController)
}
```

```
type MyCommandSet struct {
	*ccomand.CommandSet
	controller *MyService
}

func NewMyCommandSet(controller *MyService) *MyCommandSet {
	c := &MyCommandSet{}
	c.controller = controller
	c.AddCommand(c.MakeGreeting())
	return c
}

func (c *MyCommandSet) MakeGreeting() *ccomand.Command {
	return ccomand.NewCommand(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (any, error) {
			name := args.GetAsString("name")
			return c.controller.Greetings(ctx, name)
		},
	)
}
```

```
type MyService struct {
	commandSet *MyCommandSet
}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) GetCommandSet() *MyCommandSet {
	if c.commandSet == nil {
		c.commandSet = NewMyCommandSet(c)
	}

	return c.commandSet
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}
```

```
type MyCommandableCloudController struct {
	*gcpctrl.CommandableCloudFunctionController
}

func NewMyCommandableCloudController() *MyCommandableCloudController {
	c := &MyCommandableCloudController{}
	c.CommandableCloudFunctionController = gcpctrl.NewCommandableCloudFunctionController("mygroup")
	// The "service" dependency is required by all Commandable controllers
	c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	return c
}
```

```
import (
	"context"
	"net/http"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	gcpctrl "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/controllers"
	gcputil "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/utils"
	rpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
	ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type MyCommandSet struct {
	*ccomand.CommandSet
	controller *MyService
}

func NewMyCommandSet(controller *MyService) *MyCommandSet {
	c := &MyCommandSet{}
	c.controller = controller
	c.AddCommand(c.MakeGreeting())
	return c
}

func (c *MyCommandSet) MakeGreeting() *ccomand.Command {
	return ccomand.NewCommand(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (any, error) {
			name := args.GetAsString("name")
			return c.controller.Greetings(ctx, name)
		},
	)
}

type MyCommandableCloudController struct {
	*gcpctrl.CommandableCloudFunctionController
}

func NewMyCommandableCloudController() *MyCommandableCloudController {
	c := &MyCommandableCloudController{}
	c.CommandableCloudFunctionController = gcpctrl.NewCommandableCloudFunctionController("mygroup")
	// The "service" dependency is required by all Commandable controllers
	c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	return c
}

type MyCloudController struct {
	*gcpctrl.CloudFunctionController
	service *MyService
	headers map[string]string
}

func NewMyCloudController() *MyCloudController {
	c := &MyCloudController{}
	c.CloudFunctionController = gcpctrl.NewCloudFunctionController("")
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	return c
}

func (c *MyCloudController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunctionController.SetReferences(ctx, references)
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// res, err := c.DependencyResolver.GetOneRequired("service")
	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"))
	if err != nil {
		panic("Service is not found")
	}

	c.service = res.(*MyService)
}

func (c *MyCloudController) Register() {
	c.RegisterAction(
		"greetings",
		cvalid.NewObjectSchema().WithRequiredProperty("body", cvalid.NewObjectSchema().WithRequiredProperty("name", cconv.String)).Schema,
		func(res http.ResponseWriter, req *http.Request) {
			params := gcputil.CloudFunctionRequestHelper.GetParameters(req)
			name := params.GetAsStringWithDefault("name", "default name")

			result, err := c.service.Greetings(req.Context(), name)

			for key, value := range c.headers {
				res.Header().Add(key, value)
			}

			rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
		},
	)
}

type MyService struct {
	commandSet *MyCommandSet
}

func NewMyService() *MyService {
	return &MyService{}
}

func (c *MyService) GetCommandSet() *MyCommandSet {
	if c.commandSet == nil {
		c.commandSet = NewMyCommandSet(c)
	}

	return c.commandSet
}

func (c *MyService) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService) Greetings(ctx context.Context, name string) (string, error) {
	return "Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service", "1.0"), NewMyService)
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "commandable-gcp-function", "function", "1.0"), NewMyCloudController)
	return c
}

type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service    *MyService
	controller *gcpctrl.CloudFunctionController
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup service")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	// Comment out the next two lines of code when using a configuration file, uncomment to use the dependency resolver
	// c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	// c.DependencyResolver.Put(context.Background(), "controller", cref.NewDescriptor("mygroup", "controller", "commandable-gcp-function", "function", "*"))
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)
	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// serv, err := c.DependencyResolver.GetOneRequired("service")
	// Comment out the next line of code when using the dependency resolver, uncomment for configuration file
	serv, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "default", "service", "*"))
	if err != nil {
		panic("Service is not found")
	}

	// Comment out the next line of code when using a configuration file, uncomment to use the dependency resolver
	// contr, err := c.DependencyResolver.GetOneRequired("controller")
	// Comment out the next line of code when using the dependency resolver, uncomment for configuration file
	contr, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "controller", "commandable-gcp-function", "function", "*"))
	if err != nil {
		panic("Controller is not found")
	}

	c.service = serv.(*MyService)
	c.controller = contr.(*gcpctrl.CloudFunctionController)
}

```

```
import (
	"context"
	"net/http"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	gcputil "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/utils"
	rpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyService1 struct {
}

func NewMyService1() *MyService1 {
	return &MyService1{}
}

func (c *MyService1) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService1) Greetings1(ctx context.Context, name string) (string, error) {
	return "greetings1: Hello, " + name + " !", nil
}

type MyService2 struct {
}

func NewMyService2() *MyService2 {
	return &MyService2{}
}

func (c *MyService2) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService2) Greetings2(ctx context.Context, name string) (string, error) {
	return "greetings2: Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service1", "1.0"), NewMyService1)
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service2", "1.0"), NewMyService2)
	return c
}

type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service1 *MyService1
	service2 *MyService2
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	c.DependencyResolver.Put(context.Background(), "service1", cref.NewDescriptor("mygroup", "service", "default", "service1", "*"))
	c.DependencyResolver.Put(context.Background(), "service2", cref.NewDescriptor("mygroup", "service", "default", "service2", "*"))
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)

	ctrl1, err := c.DependencyResolver.GetOneRequired("service1")
	if err != nil {
		panic("service1 is not found")
	}

	ctrl2, err := c.DependencyResolver.GetOneRequired("service2")
	if err != nil {
		panic("service2 is not found")
	}

	c.service1 = ctrl1.(*MyService1)
	c.service2 = ctrl2.(*MyService2)
}

func (c *MyCloudFunction) Action1(res http.ResponseWriter, req *http.Request) {
	params := gcputil.CloudFunctionRequestHelper.GetParameters(req)

	name := params.GetAsStringWithDefault("name", "default name")
	result, err := c.service1.Greetings1(req.Context(), name)

	rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Action2(res http.ResponseWriter, req *http.Request) {
	params := gcputil.CloudFunctionRequestHelper.GetParameters(req)

	name := params.GetAsStringWithDefault("name", "default name")
	result, err := c.service2.Greetings2(req.Context(), name)

	rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Register() {
	c.RegisterAction("greetings1", nil, c.Action1)
	c.RegisterAction("greetings2", nil, c.Action2)
}
```

```
import (
	"context"
	"net/http"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	gcpcont "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/containers"
	gcpctrl "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/controllers"
	gcputil "github.com/pip-services4/pip-services4-go/pip-services4-gcp-go/utils"
	rpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyCloudFunctionConroller struct {
	*gcpctrl.CloudFunctionController
	service *MyService1

	headers map[string]string
}

func NewMyCloudFunctionController() *MyCloudFunctionConroller {
	c := &MyCloudFunctionConroller{}
	c.CloudFunctionController = gcpctrl.NewCloudFunctionController("mycontroller")
	return c
}

func (c *MyCloudFunctionConroller) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunctionController.SetReferences(ctx, references)
	res, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "*", "service1", "1.0"))
	if err != nil {
		panic("Service is not found")
	}

	c.service = res.(*MyService1)
}

func (c *MyCloudFunctionConroller) Action(res http.ResponseWriter, req *http.Request) {
	params := gcputil.CloudFunctionRequestHelper.GetParameters(req)

	name := params.GetAsStringWithDefault("name", "default name")
	result, err := c.service.Greetings1(req.Context(), name)

	rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunctionConroller) Register() {
	c.RegisterAction("greetings1", nil, c.Action)
}

type MyService1 struct {
}

func NewMyService1() *MyService1 {
	return &MyService1{}
}

func (c *MyService1) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService1) Greetings1(ctx context.Context, name string) (string, error) {
	return "Greetings from controller: Hello, " + name + " !", nil
}

type MyService2 struct {
}

func NewMyService2() *MyService2 {
	return &MyService2{}
}

func (c *MyService2) SetReferences(ctx context.Context, references cref.IReferences) {

}

func (c *MyService2) Greetings2(ctx context.Context, name string) (string, error) {
	return "Greetings from container: Hello, " + name + " !", nil
}

type MyFactory struct {
	*cbuild.Factory
}

func NewMyFactory() *MyFactory {
	c := &MyFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service1", "1.0"), NewMyService1)
	c.RegisterType(cref.NewDescriptor("mygroup", "service", "default", "service2", "1.0"), NewMyService2)
	c.RegisterType(cref.NewDescriptor("mygroup", "controller", "gcp-function", "*", "1.0"), NewMyCloudFunctionController)
	return c
}

type MyCloudFunction struct {
	*gcpcont.CloudFunction
	service *MyService2
}

func NewMyCloudFunction() *MyCloudFunction {
	c := &MyCloudFunction{}
	c.CloudFunction = gcpcont.InheritCloudFunctionWithParams(c, "mygroup", "Mygroup")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewMyFactory())
	return c
}

func (c *MyCloudFunction) SetReferences(ctx context.Context, references cref.IReferences) {
	c.CloudFunction.SetReferences(ctx, references)
	c.Counters.SetReferences(ctx, references)

	ctrl, err := references.GetOneRequired(cref.NewDescriptor("mygroup", "service", "*", "service2", "1.0"))
	if err != nil {
		panic("Service2 is not found")
	}

	c.service = ctrl.(*MyService2)
}

func (c *MyCloudFunction) Action(res http.ResponseWriter, req *http.Request) {
	params := gcputil.CloudFunctionRequestHelper.GetParameters(req)

	name := params.GetAsStringWithDefault("name", "default name")
	result, err := c.service.Greetings2(req.Context(), name)

	rpcctrl.HttpResponseSender.SendResult(res, req, result, err)
}

func (c *MyCloudFunction) Register() {
	c.RegisterAction("greetings2", nil, c.Action)
}
```

```

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Conroller
- descriptor: "mygroup:controller:gcp-function:*:1.0"

# Service 1
- descriptor: "mygroup:service:default:servicer1:1.0"

# Service 2
- descriptor: "mygroup:service:default:service2:1.0"
```

```
functions-framework --target handler --signature-type http --port 8080 --source program_name.py
```

```
curl -d '{"cmd": ""myservice.greetings1", "name": "Bob"}' -H "Content-Type: application/json" -X POST http://localhost:8080

```
