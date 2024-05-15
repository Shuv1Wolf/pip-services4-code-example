```
import (
	"context"
	"io/ioutil"
	"net/http"

	"github.com/gorilla/mux"
	cerr "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/errors"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	controller "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

func main() {
	// Instantiation
	myRestController := NewMyRestController()

	// REST controller configuration
	myRestController.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 15235,
	))

	// Connection
	myRestController.Open(context.Background())

}

type MyRestController struct {
	*controller.RestController
}

func NewMyRestController() *MyRestController {
	c := &MyRestController{}
	c.RestController = controller.InheritRestController(c)
	c.BaseRoute = "/my_service"
	return c
}

// GET
func (c *MyRestController) myPageGet(res http.ResponseWriter, req *http.Request) {
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]
	c.SendResult(res, req, result, nil)
}

// HEAD
func (c *MyRestController) myPageHead(res http.ResponseWriter, req *http.Request) {
	c.SendResult(res, req, nil, nil)
}

// POST
func (c *MyRestController) myPagePost(res http.ResponseWriter, req *http.Request) {
	traceId := c.GetTraceId(req)
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]

	var data string

	bodyBytes, bodyErr := ioutil.ReadAll(req.Body)
	_ = req.Body.Close()
	if bodyErr != nil {
		err := cerr.NewInternalError(traceId, "JSON_CNV_ERR", "Cant convert from JSON to Data").WithCause(bodyErr)
		c.SendError(res, req, err)
		return
	}

	data = string(bodyBytes)
	result = result + ", data:" + data
	c.SendResult(res, req, result, nil)
}

// PUT
func (c *MyRestController) myPagePut(res http.ResponseWriter, req *http.Request) {
	traceId := c.GetTraceId(req)
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]

	var data string

	bodyBytes, bodyErr := ioutil.ReadAll(req.Body)
	_ = req.Body.Close()
	if bodyErr != nil {
		err := cerr.NewInternalError(traceId, "JSON_CNV_ERR", "Cant convert from JSON to Data").WithCause(bodyErr)
		c.SendError(res, req, err)
		return
	}

	data = string(bodyBytes)
	result = result + ", data:" + data
	c.SendResult(res, req, result, nil)
}

// Route registration
func (c *MyRestController) Register() {
	c.RegisterRoute(
		http.MethodGet, "/my_page/{name}",
		nil,
		c.myPageGet,
	)

	c.RegisterRoute(
		http.MethodHead, "/my_page/{name}",
		nil,
		c.myPageHead,
	)

	c.RegisterRoute(
		http.MethodPost, "/my_page/{name}",
		nil,
		c.myPagePost,
	)

	c.RegisterRoute(
		http.MethodPut, "/my_page/{name}",
		nil,
		c.myPagePut,
	)
}

```

```
import (
	"context"
	"net/http"

	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	clients "github.com/pip-services4/pip-services4-go/pip-services4-http-go/clients"
)

func main() {
	// Instantiation
	client := NewMyRestClient()
	// REST client configuration
	client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 15235,
	))

	// Connection
	client.Open(context.Background())
}

type MyRestClient struct {
	*clients.RestClient
}

func NewMyRestClient() *MyRestClient {
	drc := MyRestClient{}
	drc.RestClient = clients.NewRestClient()
	drc.BaseRoute = "/my_service"
	return &drc
}

// GET
func (c *MyRestClient) GetDataGet(ctx context.Context, traceId string, name string) (result string, err error) {
	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodGet, "/my_page/"+name, params, nil)
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// HEAD
func (c *MyRestClient) GetDataHead(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodHead, "/my_page/"+name, params, nil)
	if err != nil || response == nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// POST
func (c *MyRestClient) GetDataPost(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodPost, "/my_page/"+name, params, map[string]string{"data1": "my data"})
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// PUT
func (c *MyRestClient) GetDataPut(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodPost, "/my_page/"+name, params, map[string]string{"data1": "my data"})
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}
```

```
// GET
res, err := client.GetDataGet(context.Background(), "123", "David")

```

```
// HEAD
res, err = client.GetDataHead(context.Background(), "123", "David")

```

```
// POST
res, err = client.GetDataPost(context.Background(), "123", "David")

```

```
// PUT
res, err = client.GetDataPut(context.Background(), "123", "David")

```

```
import (
	"context"
	"fmt"
	"io/ioutil"
	"net/http"

	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	clients "github.com/pip-services4/pip-services4-go/pip-services4-http-go/clients"

	"github.com/gorilla/mux"
	cerr "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/errors"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	controller "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

func main() {
	// Instantiation
	myRestController := NewMyRestController()

	// REST controller configuration
	myRestController.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 15235,
	))

	// Connection
	myRestController.Open(context.Background())
	////////////////////////////////////////////////////////////

	// Instantiation
	client := NewMyRestClient()
	// REST client configuration
	client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 15235,
	))

	// Connection
	client.Open(context.Background())

	// GET
	res, err := client.GetDataGet(context.Background(), "123", "David")
	fmt.Println(err)
	fmt.Println(res)

	// HEAD
	res, err = client.GetDataHead(context.Background(), "123", "David")
	fmt.Println(err)
	fmt.Println(res)

	// POST
	res, err = client.GetDataPost(context.Background(), "123", "David")
	fmt.Println(err)
	fmt.Println(res)

	// PUT
	res, err = client.GetDataPut(context.Background(), "123", "David")
	fmt.Println(err)
	fmt.Println(res)

	// Close REST contoller and REST client
	_ = client.Close(context.Background())
	_ = myRestController.Close(context.Background())
}

type MyRestClient struct {
	*clients.RestClient
}

func NewMyRestClient() *MyRestClient {
	drc := MyRestClient{}
	drc.RestClient = clients.NewRestClient()
	drc.BaseRoute = "/my_service"
	return &drc
}

// GET
func (c *MyRestClient) GetDataGet(ctx context.Context, traceId string, name string) (result string, err error) {
	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodGet, "/my_page/"+name, params, nil)
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// HEAD
func (c *MyRestClient) GetDataHead(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodHead, "/my_page/"+name, params, nil)
	if err != nil || response == nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// POST
func (c *MyRestClient) GetDataPost(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodPost, "/my_page/"+name, params, map[string]string{"data1": "my data"})
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

// PUT
func (c *MyRestClient) GetDataPut(ctx context.Context, traceId string, name string) (result string, err error) {

	params := cdata.NewEmptyStringValueMap()
	params.Put("message", "hello")

	response, err := c.Call(ctx, http.MethodPost, "/my_page/"+name, params, map[string]string{"data1": "my data"})
	if err != nil {
		return "", err
	}

	return clients.HandleHttpResponse[string](response, traceId)
}

////////////////////////////////////////////////////////

type MyRestController struct {
	*controller.RestController
}

func NewMyRestController() *MyRestController {
	c := &MyRestController{}
	c.RestController = controller.InheritRestController(c)
	c.BaseRoute = "/my_service"
	return c
}

// GET
func (c *MyRestController) myPageGet(res http.ResponseWriter, req *http.Request) {
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]
	c.SendResult(res, req, result, nil)
}

// HEAD
func (c *MyRestController) myPageHead(res http.ResponseWriter, req *http.Request) {
	c.SendResult(res, req, nil, nil)
}

// POST
func (c *MyRestController) myPagePost(res http.ResponseWriter, req *http.Request) {
	traceId := c.GetTraceId(req)
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]

	var data string

	bodyBytes, bodyErr := ioutil.ReadAll(req.Body)
	_ = req.Body.Close()
	if bodyErr != nil {
		err := cerr.NewInternalError(traceId, "JSON_CNV_ERR", "Cant convert from JSON to Data").WithCause(bodyErr)
		c.SendError(res, req, err)
		return
	}

	data = string(bodyBytes)
	result = result + ", data:" + data
	c.SendResult(res, req, result, nil)
}

// PUT
func (c *MyRestController) myPagePut(res http.ResponseWriter, req *http.Request) {
	traceId := c.GetTraceId(req)
	params := req.URL.Query()
	routeVars := mux.Vars(req)
	result := params.Get("message") + ", " + routeVars["name"]

	var data string

	bodyBytes, bodyErr := ioutil.ReadAll(req.Body)
	_ = req.Body.Close()
	if bodyErr != nil {
		err := cerr.NewInternalError(traceId, "JSON_CNV_ERR", "Cant convert from JSON to Data").WithCause(bodyErr)
		c.SendError(res, req, err)
		return
	}

	data = string(bodyBytes)
	result = result + ", data:" + data
	c.SendResult(res, req, result, nil)
}

// Route registration
func (c *MyRestController) Register() {
	c.RegisterRoute(
		http.MethodGet, "/my_page/{name}",
		nil,
		c.myPageGet,
	)

	c.RegisterRoute(
		http.MethodHead, "/my_page/{name}",
		nil,
		c.myPageHead,
	)

	c.RegisterRoute(
		http.MethodPost, "/my_page/{name}",
		nil,
		c.myPagePost,
	)

	c.RegisterRoute(
		http.MethodPut, "/my_page/{name}",
		nil,
		c.myPagePut,
	)
}

```
