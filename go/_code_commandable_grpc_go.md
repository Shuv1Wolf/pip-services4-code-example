```
import (
    grpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/controllers"
)

```

```
import (
    grpcclnt "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/clients"
)
```

```
type MyData struct {
	Id      string `json:"id"`
	Key     string `json:"key"`
	Content string `json:"content"`
}

func NewMyData(id string, key string, content string) *MyData {
	return &MyData{
		Id:      id,
		Key:     key,
		Content: content,
	}
}
```

```
import (
	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
)

type MyDataSchema struct {
	*cvalid.ObjectSchema
}

func NewMyDataSchema() *MyDataSchema {
	c := &MyDataSchema{}
	c.ObjectSchema = cvalid.NewObjectSchema()
	c.WithOptionalProperty("id", cconv.String)
	c.WithRequiredProperty("key", cconv.String)
	c.WithOptionalProperty("content", cconv.String)

	return c
}
```

```
import (
        cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)

var FactoryDescriptor = cref.NewDescriptor("service-mydata", "factory", "default", "default", "1.0")
var ServiceDescriptor = cref.NewDescriptor("service-mydata", "service", "default", "*", "1.0")
var CommandableGrpcControllerDescriptor = cref.NewDescriptor("service-mydata", "controller", "commandable-grpc", "*", "1.0")

type DefaultMyDataFactory struct {
	*cbuild.Factory
}

func NewDefaultMyDataFactory() *DefaultMyDataFactory {
	c := DefaultMyDataFactory{
		Factory: cbuild.NewFactory(),
	}

	c.RegisterType(ServiceDescriptor, NewMyDataService)
	c.RegisterType(CommandableGrpcControllerDescriptor, NewMyDataCommandableGrpcController)

	return &c
}
```

```
import (
	"context"
	"encoding/json"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type MyDataCommandSet struct {
	*ccomand.CommandSet
	service IMyDataService
}

func NewMyDataCommandSet(service IMyDataService) *MyDataCommandSet {
	dcs := MyDataCommandSet{}
	dcs.CommandSet = ccomand.NewCommandSet()

	dcs.service = service

	dcs.AddCommand(dcs.makePageByFilterCommand())
	dcs.AddCommand(dcs.makeGetOneByIdCommand())
	dcs.AddCommand(dcs.makeCreateCommand())
	dcs.AddCommand(dcs.makeUpdateCommand())
	dcs.AddCommand(dcs.makeDeleteByIdCommand())
	return &dcs
}

func (c *MyDataCommandSet) makePageByFilterCommand() ccomand.ICommand {
	return ccomand.NewCommand(
		"get_my_datas",
		cvalid.NewObjectSchema().WithOptionalProperty("filter", cvalid.NewFilterParamsSchema()).WithOptionalProperty("paging", cvalid.NewPagingParamsSchema()),
		func(ctx context.Context, args *cexec.Parameters) (result any, err error) {
			var filter *cquery.FilterParams
			var paging *cquery.PagingParams

			if _val, ok := args.Get("filter"); ok {
				filter = cquery.NewFilterParamsFromValue(_val)
			}
			if _val, ok := args.Get("paging"); ok {
				paging = cquery.NewPagingParamsFromValue(_val)
			}

			return c.service.GetPageByFilter("makePageByFilterCommand", filter, paging)
		},
	)
}

func (c *MyDataCommandSet) makeGetOneByIdCommand() ccomand.ICommand {
	return ccomand.NewCommand(
		"get_my_data_by_id",
		cvalid.NewObjectSchema().WithRequiredProperty("my_data_id", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (result any, err error) {
			id := args.GetAsString("my_data_id")
			return c.service.GetOneById("makeGetOneByIdCommand", id)
		},
	)
}

func (c *MyDataCommandSet) makeCreateCommand() ccomand.ICommand {
	return ccomand.NewCommand(
		"create_my_data",
		cvalid.NewObjectSchema().WithRequiredProperty("my_data", NewMyDataSchema()),
		func(ctx context.Context, args *cexec.Parameters) (result any, err error) {
			var entity MyData

			if _val, ok := args.Get("my_data"); ok {
				val, _ := json.Marshal(_val)
				json.Unmarshal(val, &entity)
			}

			return c.service.Create("makeCreateCommand", entity)
		},
	)
}

func (c *MyDataCommandSet) makeUpdateCommand() ccomand.ICommand {
	return ccomand.NewCommand(
		"update_my_data",
		cvalid.NewObjectSchema().WithRequiredProperty("my_data", NewMyDataSchema()),
		func(ctx context.Context, args *cexec.Parameters) (result any, err error) {
			var entity MyData

			if _val, ok := args.Get("my_data"); ok {
				val, _ := json.Marshal(_val)
				json.Unmarshal(val, &entity)
			}
			return c.service.Update("makeUpdateCommand", entity)
		},
	)
}

func (c *MyDataCommandSet) makeDeleteByIdCommand() ccomand.ICommand {
	return ccomand.NewCommand(
		"delete_my_data",
		cvalid.NewObjectSchema().WithRequiredProperty("my_data_id", cconv.String),
		func(ctx context.Context, args *cexec.Parameters) (result any, err error) {
			id := args.GetAsString("my_data_id")
			return c.service.DeleteById("makeDeleteByIdCommand", id)
		},
	)
}
```

```
import (
      cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type IMyDataService interface {
	GetPageByFilter(correlationId string, filter *cquery.FilterParams, paging *cquery.PagingParams) (result *cquery.DataPage[MyData], err error)
	GetOneById(correlationId string, id string) (result *MyData, err error)
	Create(correlationId string, entity MyData) (result *MyData, err error)
	Update(correlationId string, entity MyData) (result *MyData, err error)
	DeleteById(correlationId string, id string) (result *MyData, err error)
}
```

```
import (
        ccomand "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
        ckeys "github.com/pip-services4/pip-services4-go/pip-services4-data-go/keys"
        cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type MyDataService struct {
	commandSet *MyDataCommandSet
	entities   []MyData
}

func NewMyDataService() *MyDataService {
	dc := MyDataService{}
	dc.entities = make([]MyData, 0)
	return &dc
}

func (c *MyDataService) GetCommandSet() *ccomand.CommandSet {
	if c.commandSet == nil {
		c.commandSet = NewMyDataCommandSet(c)
	}
	return c.commandSet.CommandSet
}

func (c *MyDataService) GetPageByFilter(correlationId string, filter *cquery.FilterParams,
	paging *cquery.PagingParams) (items *cquery.DataPage[MyData], err error) {

	if filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}
	var key string = filter.GetAsString("key")

	if paging == nil {
		paging = cquery.NewEmptyPagingParams()
	}
	var skip int64 = paging.GetSkip(0)
	var take int64 = paging.GetTake(100)

	var result []MyData
	for i := 0; i < len(c.entities); i++ {
		var entity MyData = c.entities[i]
		if key != "" && key != entity.Key {
			continue
		}

		skip--
		if skip >= 0 {
			continue
		}

		take--
		if take < 0 {
			break
		}

		result = append(result, entity)
	}
	var total int64 = (int64)(len(result))
	return cquery.NewDataPage[MyData](result, int(total)), nil
}

func (c *MyDataService) GetOneById(correlationId string, id string) (result *MyData, err error) {
	for i := 0; i < len(c.entities); i++ {
		var entity MyData = c.entities[i]
		if id == entity.Id {
			return &entity, nil
		}
	}
	return nil, nil
}

func (c *MyDataService) Create(correlationId string, entity MyData) (result *MyData, err error) {
	if entity.Id == "" {
		entity.Id = ckeys.IdGenerator.NextLong()
	}

	c.entities = append(c.entities, entity)
	return &entity, nil
}

func (c *MyDataService) Update(correlationId string, newEntity MyData) (result *MyData, err error) {
	for index := 0; index < len(c.entities); index++ {
		var entity MyData = c.entities[index]
		if entity.Id == newEntity.Id {
			c.entities[index] = newEntity
			return &newEntity, nil

		}
	}
	return nil, nil
}

func (c *MyDataService) DeleteById(correlationId string, id string) (result *MyData, err error) {
	var entity MyData

	for i := 0; i < len(c.entities); {
		entity = c.entities[i]
		if entity.Id == id {
			if i == len(c.entities)-1 {
				c.entities = c.entities[:i]
			} else {
				c.entities = append(c.entities[:i], c.entities[i+1:]...)
			}
			return &entity, nil
		} else {
			i++
		}
	}
	return nil, nil
}
```

```
import (
        grpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/controllers"
        cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type MyDataCommandableGrpcController struct {
	*grpcctrl.CommandableGrpcController
}

func NewMyDataCommandableGrpcController() *MyDataCommandableGrpcController {
	c := &MyDataCommandableGrpcController{}
	c.CommandableGrpcController = grpcctrl.InheritCommandableGrpcController(c, "mydata")
	c.DependencyResolver.Put(context.Background(), "service", cref.NewDescriptor("service-mydata", "service", "default", "*", "*"))
	return c
}
```

```
import (
        ccont "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	grpcbuild "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/build"
)

type MyDataProcess struct {
	*ccont.ProcessContainer
}

func NewMyDataProcess() *MyDataProcess {
	c := &MyDataProcess{}
	c.ProcessContainer = ccont.NewProcessContainer("my_data", "simple my data microservice")
	c.AddFactory(NewDefaultMyDataFactory())
	c.AddFactory(grpcbuild.NewDefaultGrpcFactory())

	return c
}
```

```
---
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "mydata"
  description: "MyData microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Service
- descriptor: "service-mydata:service:default:default:1.0"

# Common GRPC endpoint
- descriptor: "pip-services:endpoint:grpc:default:1.0"
  connection:
    protocol: "http"
    host: "0.0.0.0"
    port: 8090

# Commandable GRPC endpoint version 1.0
- descriptor: "service-mydata:controller:commandable-grpc:default:1.0"
```

```
import (
        "context"
        cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)
type IMyDataClient interface {
	GetMyDatas(ctx context.Context, correlationId string, filter *cquery.FilterParams, paging *cquery.PagingParams) (result *cquery.DataPage[MyData], err error)
	GetMyDataById(ctx context.Context, correlationId string, dummyId string) (result *MyData, err error)
	CreateMyData(ctx context.Context, correlationId string, dummy MyData) (result *MyData, err error)
	UpdateMyData(ctx context.Context, correlationId string, dummy MyData) (result *MyData, err error)
	DeleteMyData(ctx context.Context, correlationId string, dummyId string) (result *MyData, err error)
}
```

```
import (
        "context"

        grpcclnt "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/clients"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
        cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type MyDataCommandableGrpcClient struct {
	*grpcclnt.CommandableGrpcClient
}

func NewMyDataCommandableGrpcClient() *MyDataCommandableGrpcClient {
	dcgc := MyDataCommandableGrpcClient{}
	dcgc.CommandableGrpcClient = grpcclnt.NewCommandableGrpcClient("mydata")
	return &dcgc
}

func (c *MyDataCommandableGrpcClient) GetMyDatas(ctx context.Context,  string, filter *cquery.FilterParams, paging *cquery.PagingParams) (result *cquery.DataPage[MyData], err error) {

	params := cdata.NewEmptyStringValueMap()
	c.AddFilterParams(params, filter)
	c.AddPagingParams(params, paging)

	response, calErr := c.CallCommand(ctx, "get_my_datas", cdata.NewAnyValueMapFromValue(params.Value()))
	if calErr != nil {
		return nil, calErr
	}

	return grpcclnt.HandleHttpResponse[*cquery.DataPage[MyData]](response)
}

func (c *MyDataCommandableGrpcClient) GetMyDataById(ctx context.Context, mydataId string) (result *MyData, err error) {

	params := cdata.NewEmptyAnyValueMap()
	params.Put("my_data_id", mydataId)

	response, calErr := c.CallCommand(ctx, "get_my_data_by_id", params)
	if calErr != nil {
		return nil, calErr
	}

	return grpcclnt.HandleHttpResponse[*MyData](response)
}

func (c *MyDataCommandableGrpcClient) CreateMyData(ctx context.Context, mydata MyData) (result *MyData, err error) {

	params := cdata.NewEmptyAnyValueMap()
	params.Put("my_data", mydata)

	response, calErr := c.CallCommand(ctx, "create_my_data", params)
	if calErr != nil {
		return nil, calErr
	}

	return grpcclnt.HandleHttpResponse[*MyData](response)
}

func (c *MyDataCommandableGrpcClient) UpdateMyData(ctx context.Context, mydata MyData) (result *MyData, err error) {

	params := cdata.NewEmptyAnyValueMap()
	params.Put("my_data", mydata)

	response, calErr := c.CallCommand(ctx, "update_my_data", params)
	if calErr != nil {
		return nil, calErr
	}

	return grpcclnt.HandleHttpResponse[*MyData](response)
}

func (c *MyDataCommandableGrpcClient) DeleteMyData(ctx context.Context, mydataId string) (result *MyData, err error) {

	params := cdata.NewEmptyAnyValueMap()
	params.Put("my_data_id", mydataId)

	response, calErr := c.CallCommand(ctx, "delete_my_data", params)
	if calErr != nil {
		return nil, calErr
	}

	return grpcclnt.HandleHttpResponse[*MyData](response)
}
```

```
import (
    "os"
    "context"
)

func main() {
	proc := NewMyDataProcess()
	proc.SetConfigPath("./config/config.yml")
	proc.Run(context.Background(), os.Args)
}
```

```
import (
        "context"
        "testing"
	pack "tst/pack"

        cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
        cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
        "github.com/stretchr/testify/assert" 
)

func TestRun(t *testing.T) {
	ctx := context.Background()
	correlationId := "123"
	// create client
	grpcConfig := cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 8090,
	)

	grpcClient := pack.NewMyDataCommandableGrpcClient()
	grpcClient.Configure(ctx, grpcConfig)
	grpcClient.SetReferences(ctx, cref.NewReferences(ctx, make([]any, 0)))
	err := grpcClient.Open(ctx, correlationId)
	assert.Nil(t, err)
	// simple data
	data1 := pack.MyData{Id: "1", Key: "0005", Content: "any content 1"}
	data2 := pack.MyData{Id: "2", Key: "0010", Content: "any content 2"}

	// using the client
	res, err := grpcClient.CreateMyData(ctx, correlationId, data1)
	assert.Nil(t, err)
	assert.Equal(t, res.Id, data1.Id)

	res, err = grpcClient.CreateMyData(ctx, correlationId, data2)
	assert.Nil(t, err)
	assert.Equal(t, res.Id, data2.Id)

	resPage, err := grpcClient.GetMyDatas(ctx, correlationId, nil, nil)
	assert.Nil(t, err)
	assert.Equal(t, len(resPage.Data), 2)

	res, err = grpcClient.DeleteMyData(ctx, correlationId, data2.Id)
	assert.Nil(t, err)
	assert.Equal(t, res.Id, data2.Id)

	res, err = grpcClient.GetMyDataById(ctx, correlationId, data2.Id)
	assert.Nil(t, err)
	assert.Nil(t, res)
}
```
