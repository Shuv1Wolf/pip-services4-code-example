### Step 1. Environment setup

```
go version
```

### Step 2. Project setup

```
go mod init quickstart
```

**/go.mod**

```
module quickstart

go 1.22

require (
	github.com/pip-services4/pip-services4-go/pip-services4-components-go v0.0.1-2
	github.com/pip-services4/pip-services4-go/pip-services4-container-go v0.0.1-3
)

require (
	github.com/pip-services4/pip-services4-go/pip-services4-config-go v0.0.0-20240325121312-3b0195749a25 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-data-go v0.0.1-2 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-expressions-go v0.0.1-2 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-logic-go v0.0.1-3 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-observability-go v0.0.1-3 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-rpc-go v0.0.0-20240304141352-928143cb0946 // indirect
	github.com/rs/cors v1.9.0 // indirect
	goji.io v2.0.2+incompatible // indirect
	gopkg.in/yaml.v2 v2.4.0 // indirect
)

require (
	github.com/google/uuid v1.6.0 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-commons-go v0.0.1-2 // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-http-go v0.0.1-4
)

```

```
go get -u
```

### Step 3. Service

```go
func (c *HelloWorldService) Greeting(ctx context.Context, name string) (result string, err error) {
	if name == "" {
		name = c.defaultName
	}
	return "Hello, " + name + "!", nil
}
```

```
func (c *HelloWorldService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.defaultName = config.GetAsStringWithDefault("default_name", c.defaultName)
}
```

```yaml
# Service
- descriptor: "hello-world:service:default:default:1.0"
  default_name: "World"
```

**/HelloWorldService.go**

```go
package quickstart

import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type HelloWorldService struct {
	defaultName string
}

func NewHelloWorldService() *HelloWorldService {
	c := HelloWorldService{}
	c.defaultName = "Pip User"
	return &c
}

func (c *HelloWorldService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.defaultName = config.GetAsStringWithDefault("default_name", c.defaultName)
}

func (c *HelloWorldService) Greeting(ctx context.Context, name string) (result string, err error) {
	if name == "" {
		name = c.defaultName
	}
	return "Hello, " + name + "!", nil
}

```

### Step 4. REST controller

```go
type HelloWorldRestController struct {
	*rpc.RestController
	service *HelloWorldService
}
```

```go
func (c *HelloWorldRestController) greeting(res http.ResponseWriter, req *http.Request) {
	name := req.URL.Query().Get("name")
	result, err := c.service.Greeting(req.Context(), name)
	c.SendResult(res, req, result, err)
}

func (c *HelloWorldRestController) Register() {
	c.RegisterRoute("get", "/greeting", nil, c.greeting)
}
```

```go
func NewHelloWorldRestController() *HelloWorldRestController {
	c := &HelloWorldRestController{}
	c.RestController = rpc.InheritRestController(c)
	c.BaseRoute = "/hello_world"
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("hello-world", "service", "*", "*", "1.0"))
	return c
}
```

**/HelloWorldRestController.go**

```go
package quickstart

import (
	"context"
	"net/http"

	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	rpc "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type HelloWorldRestController struct {
	*rpc.RestController
	service *HelloWorldService
}

func NewHelloWorldRestController() *HelloWorldRestController {
	c := &HelloWorldRestController{}
	c.RestController = rpc.InheritRestController(c)
	c.BaseRoute = "/hello_world"
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("hello-world", "service", "*", "*", "1.0"))
	return c
}

func (c *HelloWorldRestController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.RestController.SetReferences(ctx, references)
	depRes, depErr := c.DependencyResolver.GetOneRequired("service")
	if depErr == nil && depRes != nil {
		c.service = depRes.(*HelloWorldService)
	}
}

func (c *HelloWorldRestController) greeting(res http.ResponseWriter, req *http.Request) {
	name := req.URL.Query().Get("name")
	result, err := c.service.Greeting(req.Context(), name)
	c.SendResult(res, req, result, err)
}

func (c *HelloWorldRestController) Register() {
	c.RegisterRoute("get", "/greeting", nil, c.greeting)
}

```

### Step 5. Component factory

```go
type HelloWorldControllerFactory struct {
	*cbuild.Factory
}
```

```go
func NewHelloWorldControllerFactory() *HelloWorldControllerFactory {
	c := HelloWorldControllerFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(
		cref.NewDescriptor("hello-world", "service", "default", "*", "1.0"),
		NewHelloWorldService,
	)
	c.RegisterType(
		cref.NewDescriptor("hello-world", "controller", "http", "*", "1.0"),
		NewHelloWorldRestController,
	)
	return &c
}
```

**‍/HelloWorldControllerFactory.go**

```go
package quickstart

import (
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type HelloWorldControllerFactory struct {
	*cbuild.Factory
}

func NewHelloWorldControllerFactory() *HelloWorldControllerFactory {
	c := HelloWorldControllerFactory{}
	c.Factory = cbuild.NewFactory()
	c.RegisterType(
		cref.NewDescriptor("hello-world", "service", "default", "*", "1.0"),
		NewHelloWorldService,
	)
	c.RegisterType(
		cref.NewDescriptor("hello-world", "controller", "http", "*", "1.0"),
		NewHelloWorldRestController,
	)
	return &c
}

```

### Step 6. Container

**‍/HelloWorldProcess.go**

```go
package quickstart

import (
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	rbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
)

type HelloWorldProcess struct {
	*cproc.ProcessContainer
}

func NewHelloWorldProcess() *HelloWorldProcess {
	c := HelloWorldProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("hello-world", "HelloWorld microservice")
	c.SetConfigPath("./config.yaml")
	c.AddFactory(NewHelloWorldControllerFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())
	return &c
}

```

**‍/config.yaml**

```yaml
---
# Container context
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "hello-world"
  description: "HelloWorld microservice"
   
# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"
   
# Performance counter that post values to log
- descriptor: "pip-services:counters:log:default:1.0"
   
# Service
- descriptor: "hello-world:service:default:default:1.0"
  default_name: "World"
   
# Shared HTTP Endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: 8080
   
    # HTTP controller V1
- descriptor: "hello-world:controller:http:default:1.0"
   
# Heartbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"
   
# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"


```

### Step 7. Run and test the microservice

**/bin/main.go**

```go
package main

import (
	"context"
	"os"
	"quickstart"
)

func main() {
	proc := quickstart.NewHelloWorldProcess()
	proc.Run(context.Background(), os.Args)
}

```

```
go run ./bin/run.go
```

```
[hello-world:INFO:2024-04-26T07:03:30Z] Press Control-C to stop the microservice...

Created component pip-services:context-info:default:default:1.0
Created component pip-services:logger:console:default:1.0
Created component pip-services:counters:log:default:1.0
Created component hello-world:service:default:default:1.0
Created component pip-services:endpoint:http:default:1.0
Created component hello-world:controller:http:default:1.0
Created component pip-services:heartbeat-controller:http:default:1.0
Created component pip-services:status-controller:http:default:1.0
[hello-world:DEBUG:2024-04-26T07:03:30Z] Opened REST service at http://0.0.0.0:8080

[hello-world:INFO:2024-04-26T07:03:30Z] Container hello-world started
```
