```
import cntrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
```

```
import (
	"context"
	"net/http"
	"os"

	"github.com/gorilla/mux"
	config "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cntrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyRestController struct {
	*cntrl.RestController
}

func NewMyRestController() *MyRestController {
	c := &MyRestController{}
	c.RestController = cntrl.InheritRestController(c)
	c.BaseRoute = "/my_service"
	return c
}

func (c *MyRestController) myPage(res http.ResponseWriter, req *http.Request) {
	vars := mux.Vars(req)

	name := req.URL.Query().Get("message")
	message := vars["name"]
	result := message + ", " + name

	c.SendResult(res, req, result, nil)
}

func (c *MyRestController) Register() {
	c.RegisterRoute("GET", "/my_page/{name}", nil, c.myPage)
}
```

```
myRestController := NewMyRestController()

myRestController.Configure(context.Background(), config.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 15239,
))

_ = myRestController.Open(context.Background())
```

```
import (
	"context"
	"net/http"
	"os"

	"github.com/gorilla/mux"
	config "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cntrl "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type MyRestController struct {
	*cntrl.RestController
}

func NewMyRestController() *MyRestController {
	c := &MyRestController{}
	c.RestController = cntrl.InheritRestController(c)
	c.BaseRoute = "/my_service"
	return c
}

func (c *MyRestController) myPage(res http.ResponseWriter, req *http.Request) {
	vars := mux.Vars(req)

	name := req.URL.Query().Get("message")
	message := vars["name"]
	result := message + ", " + name

	c.SendResult(res, req, result, nil)
}

func (c *MyRestController) Register() {
	c.RegisterRoute("GET", "/my_page/{name}", nil, c.myPage)
}

func main() {
	myRestController := NewMyRestController()

	myRestController.Configure(context.Background(), config.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 15239,
	))

	_ = myRestController.Open(context.Background())

	os.Stdin.Read(make([]byte, 1)) // wait for close
}
```

```
http://localhost:15239/my_service/my_page/John?message=hello
```
