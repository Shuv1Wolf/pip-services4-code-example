Pre-requisites
```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-commons-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-http-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-mysql-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-postgres-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-data-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-container-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-components-go@latest

go get -u github.com/pip-services4/pip-services4-go/pip-services4-mysql-go@latest
```

Data object
```
type MyFriend struct {
	Id   string `bson:"_id" json:"id"`
	Type string `bson:"type" json:"type"`
	Name string `bson:"name" json:"name"`
}

func (d *MyFriend) SetId(id string) {
	d.Id = id
}

func (d MyFriend) GetId() string {
	return d.Id
}

func (d MyFriend) Clone() MyFriend {
	return MyFriend{
		Id:   d.Id,
		Type: d.Type,
		Name: d.Name,
	}
}


```

Tier 1: Presentation layer or view
```
import (
	"context"
	"net/http"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cservices "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type HelloFriendRestController struct {
	*cservices.RestController
	service *HelloFriendService
}

func NewHelloFriendRestController() *HelloFriendRestController {
	c := &HelloFriendRestController{}
	c.RestController = cservices.InheritRestController(c)
	c.BaseRoute = "/hello_friend"
	return c
}

func (c *HelloFriendRestController) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.RestController.Configure(ctx, config)
}

func (c *HelloFriendRestController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.RestController.SetReferences(ctx, references)
	res, err := references.GetOneRequired(cref.NewDescriptor("hello-friend", "service", "*", "*", "1.0"))
	if err != nil {
		panic(err)
	}

	c.service = res.(*HelloFriendService)
}

func (c *HelloFriendRestController) Greeting(res http.ResponseWriter, req *http.Request) {
	result, err := c.service.Greeting(req.Context())
	c.SendResult(res, req, result, err)
}

func (c *HelloFriendRestController) Create(res http.ResponseWriter, req *http.Request) {
	friend := MyFriend{Id: "0", Type: "New type", Name: "New name"}
	c.SendResult(res, req, friend, nil)
}

func (c *HelloFriendRestController) Register() {
	c.RegisterRoute("GET", "/greeting", nil, c.Greeting)
	c.RegisterRoute("GET", "/greeting_create", nil, c.Create)
}
```

Tier 2: Application layer or controller
```
type HelloFriendService struct {
	defaultName string
	persistence *HelloFriendPersistence
}

func NewHelloFriendService() *HelloFriendService {
	c := &HelloFriendService{}
	c.defaultName = "Pip User"
	return c
}

func (c *HelloFriendService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.defaultName = config.GetAsStringWithDefault("default_name", c.defaultName)
}

func (c *HelloFriendService) SetReferences(ctx context.Context, references cref.IReferences) {
	res, err := references.GetOneRequired(cref.NewDescriptor("hello-friend", "persistence", "*", "*", "1.0"))
	if err != nil {
		panic(err)
	}

	c.persistence = res.(*HelloFriendPersistence)
}

func (c *HelloFriendService) Greeting(ctx context.Context) (string, error) {
	filter := cquery.NewFilterParamsFromTuples("type", "friend")
	selectedFilter, err := c.persistence.GetOneRandom(ctx, "123", *filter)
	if err != nil {
		return "", err
	}

	return "Hello, " + selectedFilter.Name + "!", nil
}

func (c *HelloFriendService) Create(ctx context.Context, correlationId string, item MyFriend) (MyFriend, error) {
	return c.persistence.Create(ctx, item)
}
```

Tier 3: Data layer or persistence layer
```
import (
	mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type HelloFriendPersistence struct {
	*mysqlpersist.IdentifiableMySqlPersistence[MyFriend, string]
}

func NewHelloFriendPersistence() *HelloFriendPersistence {
	c := &HelloFriendPersistence{}
	c.IdentifiableMySqlPersistence = mysqlpersist.InheritIdentifiableMySqlPersistence[MyFriend, string](c, "myfriends4")
	return c
}

func (c *HelloFriendPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE `" + c.TableName + "` (id VARCHAR(32) PRIMARY KEY, `type` VARCHAR(50), `name` TEXT)")
}

func (c *HelloFriendPersistence) composeFilter(filter cquery.FilterParams) string {
	typee, typeOk := filter.GetAsNullableString("type")
	name, nameOk := filter.GetAsNullableString("name")

	filterObj := ""
	if typeOk && typee != "" {
		filterObj += "`type`='" + typee + "'"
	}
	if nameOk && name != "" {
		filterObj += "`name`='" + name + "'"
	}

	return filterObj
```

Containerization
```
import (
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)

var HttpControllerDescriptor = cref.NewDescriptor("hello-friend", "controller", "http", "*", "1.0") // View
var ServiceDescriptor = cref.NewDescriptor("hello-friend", "service", "default", "*", "1.0")        // Service
var PersistenceDescriptor = cref.NewDescriptor("hello-friend", "persistence", "mysql", "*", "1.0")  // Persistence

type HelloFriendServiceFactory struct {
	*cbuild.Factory
}

func NewHelloFriendServiceFactory() *HelloFriendServiceFactory {
	c := &HelloFriendServiceFactory{}
	c.Factory = cbuild.NewFactory()

	c.RegisterType(HttpControllerDescriptor, NewHelloFriendRestController) // View
	c.RegisterType(ServiceDescriptor, NewHelloFriendService)               // Service
	c.RegisterType(PersistenceDescriptor, NewHelloFriendPersistence)       // Persistence

	return c
}
```

Locator pattern: config file
```
---
# Container context
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "hello-friend"
  description: "HelloFriend microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Performance counter that post values to log
- descriptor: "pip-services:counters:log:default:1.0"

# Service
- descriptor: "hello-friend:service:default:default:1.0"
  default_name: "Friend"

# Shared HTTP Endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: 8085

# HTTP controller V1
- descriptor: "hello-friend:controller:http:default:1.0"

# Heartbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"

# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"

# Persistnece - MySQL
- descriptor: "hello-friend:persistence:mysql:default:1.0"
  connection:
    host: 'localhost'
    port: '3306'
    database: 'pip'
  credential:
    username: 'root'
    password: ''

```

Process container
```
import (
	rbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
)

type HelloFriendProcess struct {
	*cproc.ProcessContainer
}

func NewHelloFriendProcess() *HelloFriendProcess {
	c := &HelloFriendProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("hello-friend", "HelloFriend microservice")
	c.AddFactory(NewHelloFriendServiceFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())

	return c
}
```

Running the app
```
func main() {
	proc := NewHelloFriendProcess()
	proc.SetConfigPath("./config.yml")
	proc.Run(context.Background(), os.Args)
}
```

Complete code
```
package main

import (
	"context"
	"net/http"
	"os"

	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	rbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	cservices "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
	mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
)

type MyFriend struct {
	Id   string `bson:"_id" json:"id"`
	Type string `bson:"type" json:"type"`
	Name string `bson:"name" json:"name"`
}

func (d *MyFriend) SetId(id string) {
	d.Id = id
}

func (d MyFriend) GetId() string {
	return d.Id
}

func (d MyFriend) Clone() MyFriend {
	return MyFriend{
		Id:   d.Id,
		Type: d.Type,
		Name: d.Name,
	}
}

type HelloFriendRestController struct {
	*cservices.RestController
	service *HelloFriendService
}

func NewHelloFriendRestController() *HelloFriendRestController {
	c := &HelloFriendRestController{}
	c.RestController = cservices.InheritRestController(c)
	c.BaseRoute = "/hello_friend"
	return c
}

func (c *HelloFriendRestController) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.RestController.Configure(ctx, config)
}

func (c *HelloFriendRestController) SetReferences(ctx context.Context, references cref.IReferences) {
	c.RestController.SetReferences(ctx, references)
	res, err := references.GetOneRequired(cref.NewDescriptor("hello-friend", "service", "*", "*", "1.0"))
	if err != nil {
		panic(err)
	}

	c.service = res.(*HelloFriendService)
}

func (c *HelloFriendRestController) Greeting(res http.ResponseWriter, req *http.Request) {
	result, err := c.service.Greeting(req.Context())
	c.SendResult(res, req, result, err)
}

func (c *HelloFriendRestController) Create(res http.ResponseWriter, req *http.Request) {
	friend := MyFriend{Id: "0", Type: "New type", Name: "New name"}
	c.SendResult(res, req, friend, nil)
}

func (c *HelloFriendRestController) Register() {
	c.RegisterRoute("GET", "/greeting", nil, c.Greeting)
	c.RegisterRoute("GET", "/greeting_create", nil, c.Create)
}


type HelloFriendService struct {
	defaultName string
	persistence *HelloFriendPersistence
}

func NewHelloFriendService() *HelloFriendService {
	c := &HelloFriendService{}
	c.defaultName = "Pip User"
	return c
}

func (c *HelloFriendService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.defaultName = config.GetAsStringWithDefault("default_name", c.defaultName)
}

func (c *HelloFriendService) SetReferences(ctx context.Context, references cref.IReferences) {
	res, err := references.GetOneRequired(cref.NewDescriptor("hello-friend", "persistence", "*", "*", "1.0"))
	if err != nil {
		panic(err)
	}

	c.persistence = res.(*HelloFriendPersistence)
}

func (c *HelloFriendService) Greeting(ctx context.Context) (string, error) {
	filter := cquery.NewFilterParamsFromTuples("type", "friend")
	selectedFilter, err := c.persistence.GetOneRandom(ctx, "123", *filter)
	if err != nil {
		return "", err
	}

	return "Hello, " + selectedFilter.Name + "!", nil
}

func (c *HelloFriendService) Create(ctx context.Context, correlationId string, item MyFriend) (MyFriend, error) {
	return c.persistence.Create(ctx, item)
}


type HelloFriendPersistence struct {
	*mysqlpersist.IdentifiableMySqlPersistence[MyFriend, string]
}

func NewHelloFriendPersistence() *HelloFriendPersistence {
	c := &HelloFriendPersistence{}
	c.IdentifiableMySqlPersistence = mysqlpersist.InheritIdentifiableMySqlPersistence[MyFriend, string](c, "myfriends4")
	return c
}

func (c *HelloFriendPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE `" + c.TableName + "` (id VARCHAR(32) PRIMARY KEY, `type` VARCHAR(50), `name` TEXT)")
}

func (c *HelloFriendPersistence) composeFilter(filter cquery.FilterParams) string {
	typee, typeOk := filter.GetAsNullableString("type")
	name, nameOk := filter.GetAsNullableString("name")

	filterObj := ""
	if typeOk && typee != "" {
		filterObj += "`type`='" + typee + "'"
	}
	if nameOk && name != "" {
		filterObj += "`name`='" + name + "'"
	}

	return filterObj
}

func (c *HelloFriendPersistence) GetOneRandom(ctx context.Context, correlationId string, filter cquery.FilterParams) (item MyFriend, err error) {
	return c.MySqlPersistence.GetOneRandom(ctx, c.composeFilter(filter))
}


var HttpControllerDescriptor = cref.NewDescriptor("hello-friend", "controller", "http", "*", "1.0") // View
var ServiceDescriptor = cref.NewDescriptor("hello-friend", "service", "default", "*", "1.0")        // Controller
var PersistenceDescriptor = cref.NewDescriptor("hello-friend", "persistence", "mysql", "*", "1.0")  // Persistence

type HelloFriendServiceFactory struct {
	*cbuild.Factory
}

func NewHelloFriendServiceFactory() *HelloFriendServiceFactory {
	c := &HelloFriendServiceFactory{}
	c.Factory = cbuild.NewFactory()

	c.RegisterType(HttpControllerDescriptor, NewHelloFriendRestController) // View
	c.RegisterType(ServiceDescriptor, NewHelloFriendService)               // Controller
	c.RegisterType(PersistenceDescriptor, NewHelloFriendPersistence)       // Persistence

	return c
}


type HelloFriendProcess struct {
	*cproc.ProcessContainer
}

func NewHelloFriendProcess() *HelloFriendProcess {
	c := &HelloFriendProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("hello-friend", "HelloFriend microservice")
	c.AddFactory(NewHelloFriendServiceFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())

	return c
}

func main() {
	proc := NewHelloFriendProcess()
	proc.SetConfigPath("./config.yml")
	proc.Run(context.Background(), os.Args)
}

```
