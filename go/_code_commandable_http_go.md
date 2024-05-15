```
import (
	ctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)
```

```
import (
	"context"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type FriendsCommandSet struct {
	*ccomand.CommandSet
	service HelloFriendService
}

func NewFriendsCommandSet(service HelloFriendService) *FriendsCommandSet {
	c := FriendsCommandSet{}
	c.service = service
	c.CommandSet = ccomand.NewCommandSet()
	c.AddCommand(c.makeGreeting())
	return &c
}

func (c *FriendsCommandSet) makeGreeting() ccomand.ICommand {
	return ccomand.NewCommand(
		"greeting",
		cvalid.NewObjectSchema().
			WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			name := args.GetAsString("name")
			return c.service.Greeting(name), nil
		},
	)
}
```

```
import (
        ctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type FriendCommandableHttpController struct {
	*ctrl.CommandableHttpController 
}

func NewFriendCommandableHttpController () *FriendCommandableHttpController  {
	c := &FriendCommandableHttpController {}
	c.CommandableHttpController  = ctrl.InheritCommandableHttpController(c, "commandable_hello_friend")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("hello-friend", "service", "*", "*", "*"))
	return c
}
```

```
import (
        ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
        cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type HelloFriendService struct {
	commandSet  *FriendsCommandSet
	defaultName string
}

func NewHelloFriendService() *HelloFriendService {
	c := HelloFriendService{}
	c.defaultName = "World"
	return &c
}

func (c *HelloFriendService) Configure(ctx context.Context,  config *cconf.ConfigParams) {
	// You can read configuration parameters here...
}


func (c *HelloFriendService) GetCommandSet() *ccomand.CommandSet {
	if c.commandSet == nil {
		c.commandSet = NewFriendsCommandSet(*c)
	}
	return c.commandSet.CommandSet
}

func (c *HelloFriendService) Greeting(name string) string {
	if name != "" {
		return "Hello, " + name + " !"
	}
	return "Hello, " + c.defaultName + " !"
}
```

```
import (
        cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)

type HelloFriendServiceFactory struct {
	*cbuild.Factory
}

func NewHelloFriendServiceFactory() *HelloFriendServiceFactory {
	c := HelloFriendServiceFactory{}
	c.Factory = cbuild.NewFactory()

	commandableHttpControllerDescriptor := cref.NewDescriptor("hello-friend", "controller", "commandable-http", "*", "1.0") 
	serviceDescriptor := cref.NewDescriptor("hello-friend", "service", "default", "*", "1.0")  

	c.RegisterType(commandableHttpControllerDescriptor, NewFriendCommandableHttpController)
	c.RegisterType(serviceDescriptor, NewHelloFriendService)

	return &c
}
```

```
import (
        sbuild "github.com/pip-services4/pip-services4-go/pip-services4-swagger-go/build"
        cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
        hbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
)

type HelloFriendProcess struct {
	*cproc.ProcessContainer
}

func NewHelloFriendProcess() *HelloFriendProcess {
	c := &HelloFriendProcess{
		ProcessContainer: cproc.NewProcessContainer("Hellow", "Hello friend microservice"),
	}

	c.SetConfigPath("./config.yaml")

	c.AddFactory(NewHelloFriendServiceFactory())
	c.AddFactory(hbuild.NewDefaultHttpFactory())
	c.AddFactory(sbuild.NewDefaultSwaggerFactory())
	return c
}
```

```
import (
	"os"
)

func main() {
	proc := NewHelloFriendProcess()
	proc.Run(os.Args)
}
```

```
import clnt "github.com/pip-services4/pip-services4-go/pip-services4-http-go/clients"
```

```
type MyCommandableHttpClient struct {
	*clnt.CommandableHttpClient
}

func NewMyCommandableHttpClient(baseRoute string) *MyCommandableHttpClient {
	c := MyCommandableHttpClient{}
	c.CommandableHttpClient = clnt.NewCommandableHttpClient(baseRoute)
	return &c
}

func (c *MyCommandableHttpClient) Greeting(ctx context.Context, correlationId string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("name", "Peter")

	res, calErr := c.CallCommand(context.Background(), "greeting", cdata.NewAnyValueMapFromValue(params.Value()))
	if calErr != nil {
		return "", calErr
	}

	return clnt.HandleHttpResponse[string](res, correlationId)
}
```

```
import (
        "context"
	"fmt"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

client := NewMyCommandableHttpClient("commandable_hello_friend")
client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 8080,
))
client.Open(context.Background())
defer client.Close(context.Background())
```

```
data, _ := client.Greeting(context.Background(), "123") // Returns 'Hello, Peter !'
fmt.Println(data)
```

```
import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	postBody, _ := json.Marshal(map[string]string{
		"name": "Peter",
	})

	responseBody := bytes.NewBuffer(postBody)

	resp, _ := http.Post("http://localhost:8080/commandable_hello_friend/greeting", "application/json", responseBody)
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	sb := string(body)
	fmt.Println(sb) // Returns '"Hello, Cosme !"'
}
```

```
import (
	"context"
	"os"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	hbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	ctrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
	ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
	sbuild "github.com/pip-services4/pip-services4-go/pip-services4-swagger-go/build"
)

func main() {
	proc := NewHelloFriendProcess()
	proc.Run(context.Background(), os.Args)
}

type HelloFriendProcess struct {
	*cproc.ProcessContainer
}

func NewHelloFriendProcess() *HelloFriendProcess {
	c := &HelloFriendProcess{
		ProcessContainer: cproc.NewProcessContainer("Hellow", "Hello friend microservice"),
	}

	c.SetConfigPath("./config.yaml")

	c.AddFactory(NewHelloFriendServiceFactory())
	c.AddFactory(hbuild.NewDefaultHttpFactory())
	c.AddFactory(sbuild.NewDefaultSwaggerFactory())
	return c
}

type HelloFriendServiceFactory struct {
	*cbuild.Factory
}

func NewHelloFriendServiceFactory() *HelloFriendServiceFactory {
	c := HelloFriendServiceFactory{}
	c.Factory = cbuild.NewFactory()

	commandableHttpControllerDescriptor := cref.NewDescriptor("hello-friend", "controller", "commandable-http", "*", "1.0")
	serviceDescriptor := cref.NewDescriptor("hello-friend", "service", "default", "*", "1.0")

	c.RegisterType(commandableHttpControllerDescriptor, NewFriendCommandableHttpController)
	c.RegisterType(serviceDescriptor, NewHelloFriendService)

	return &c
}

type HelloFriendService struct {
	commandSet  *FriendsCommandSet
	defaultName string
}

func NewHelloFriendService() *HelloFriendService {
	c := HelloFriendService{}
	c.defaultName = "World"
	return &c
}

func (c *HelloFriendService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// You can read configuration parameters here...
}

func (c *HelloFriendService) GetCommandSet() *ccomand.CommandSet {
	if c.commandSet == nil {
		c.commandSet = NewFriendsCommandSet(*c)
	}
	return c.commandSet.CommandSet
}

func (c *HelloFriendService) Greeting(name string) string {
	if name != "" {
		return "Hello, " + name + " !"
	}
	return "Hello, " + c.defaultName + " !"
}

type FriendCommandableHttpController struct {
	*ctrl.CommandableHttpController
}

func NewFriendCommandableHttpController() *FriendCommandableHttpController {
	c := &FriendCommandableHttpController{}
	c.CommandableHttpController = ctrl.InheritCommandableHttpController(c, "commandable_hello_friend")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("hello-friend", "service", "*", "*", "*"))
	return c
}

type FriendsCommandSet struct {
	*ccomand.CommandSet
	service HelloFriendService
}

func NewFriendsCommandSet(service HelloFriendService) *FriendsCommandSet {
	c := FriendsCommandSet{}
	c.service = service
	c.CommandSet = ccomand.NewCommandSet()
	c.AddCommand(c.makeGreeting())
	return &c
}

func (c *FriendsCommandSet) makeGreeting() ccomand.ICommand {
	return ccomand.NewCommand(
		"greeting",
		cvalid.NewObjectSchema().
			WithRequiredProperty("name", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			name := args.GetAsString("name")
			return c.service.Greeting(name), nil
		},
	)
}
```

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
    port: 8080
   
# Commandable HTTP controller
- descriptor: "hello-friend:controller:commandable-http:default:1.0"
  swagger:
    enable: true
    auto: true
    route: swagger
    name: Friends service
    description: Commandable REST API
  
# Heartbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"
   
# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"

# Swagger controller
- descriptor: "pip-services:swagger-controller:http:default:1.0"

```

```
import (
	"context"
	"fmt"

	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	clnt "github.com/pip-services4/pip-services4-go/pip-services4-http-go/clients"
)

func main() {
	client := NewMyCommandableHttpClient("commandable_hello_friend")
	client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 8080,
	))
	client.Open(context.Background())
	defer client.Close(context.Background())
	data, _ := client.Greeting(context.Background(), "123") // Returns 'Hello, Peter !'
	fmt.Println(data)
}

type MyCommandableHttpClient struct {
	*clnt.CommandableHttpClient
}

func NewMyCommandableHttpClient(baseRoute string) *MyCommandableHttpClient {
	c := MyCommandableHttpClient{}
	c.CommandableHttpClient = clnt.NewCommandableHttpClient(baseRoute)
	return &c
}

func (c *MyCommandableHttpClient) Greeting(ctx context.Context, correlationId string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("name", "Peter")

	res, calErr := c.CallCommand(context.Background(), "greeting", cdata.NewAnyValueMapFromValue(params.Value()))
	if calErr != nil {
		return "", calErr
	}

	return clnt.HandleHttpResponse[string](res, correlationId)
}
```
