```go
// Implements IConfigurable, IReferenceable, IUnreferenceable, IOpenable, IExecutable, INotifiable, ICleanable
type MyComponent struct{}

func (c *MyComponent) NewMyComponent() { /* Initialize the component */ }

func (c *MyComponent) Configure(ctx context.Context, config *cconf.ConfigParams) { /* configure the component */ }

func (c *MyComponent) SetReferences(ctx context.Context, refs cref.IReferences) { /* set component dependencies */ }

func (c *MyComponent) UnsetReferences() { /* unset component references */ }

func (c *MyComponent) IsOpen() bool { /* return the component open state */ }

func (c *MyComponent) Open(ctx context.Context, correlationId string) error { /* open the component */ }

func (c *MyComponent) Close(ctx context.Context, correlationId string) error { /* close the component */ }

func (c *MyComponent) Execute(ctx context.Context, correlationId string, args *crun.Parameters) error { /* execute the component transaction */
}

func (c *MyComponent) Notify(ctx context.Context, correlationId string, args *crun.Parameters) { /* notify the component about events */
}

func (c *MyComponent) Clear(ctx context.Context, correlationId string) error { /* clear the component state */ }
```

Containers and their configurations

```
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "pip-service-data"
  description: "Entities data microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Performance log counters
- descriptor: "pip-services:counters:log:default:1.0"

{{#if MONGO_ENABLED}}
# MongoDb persistence
- descriptor: "pip-service-data:persistence:mongodb:default:1.0"
  connection:
    uri: {{MONGO_SERVICE_URI}}
    host: {{MONGO_SERVICE_HOST}}{{#unless MONGO_SERVICE_HOST}}"localhost"{{/unless}}
    port: {{MONGO_SERVICE_PORT}}{{#unless MONGO_SERVICE_PORT}}27017{{/unless}}
    database: {{MONGO_DB}}{{#unless MONGO_DB}}"test"{{/unless}}
  credential:
    username: {{MONGO_USER}}
    password: {{MONGO_PASS}}
{{/if}}

{{#unless MONGO_ENABLED}}
# Default to in-memory persistence, if nothing is set
- descriptor: "pip-service-data:persistence:memory:default:1.0"
{{/unless}}

# Service
- descriptor: "pip-service-data:service:default:default:1.0"

{{#if HTTP_ENABLED}}
# Common HTTP endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{HTTP_PORT}}{{#unless HTTP_PORT}}8080{{/unless}}

# HTTP controller version 1.0
- descriptor: "pip-service-data:controller:http:default:1.0"
  swagger:
    enable: true

# Swagger service
- descriptor: "pip-services:swagger-controller:http:default:1.0"
{{/if}}

{{#if GRPC_ENABLED}}
# Common GRPC endpoint
- descriptor: "pip-services:endpoint:grpc:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{GRPC_PORT}}{{#unless GRPC_PORT}}8090{{/unless}}

# GRPC controller version 1.0
- descriptor: "pip-service-data:controller:grpc:default:1.0"
{{/if}}
```

Component factories

```
type MyProcess struct {
	*cproc.ProcessContainer
}

func NewMyProcess() *MyProcess {
	c := MyProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("mymicroservice", "Sample microservice container")
	//c.AddFactory(factory.NewMyComponentFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())
	c.AddFactory(swagger.NewDefaultSwaggerFactory())
	c.AddFactory(gbuild.NewDefaultGrpcFactory())
	return &c
}
```
