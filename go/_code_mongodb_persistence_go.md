```
package persistence

import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cmongo "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
	"go.mongodb.org/mongo-driver/bson"
)

type BeaconsMongoPersistence struct {
	cmongo.IdentifiableMongoDbPersistence[data1.BeaconV1, string]
}

func NewBeaconsMongoPersistence() *BeaconsMongoPersistence {
	c := &BeaconsMongoPersistence{}
	c.IdentifiableMongoDbPersistence = *cmongo.InheritIdentifiableMongoDbPersistence[data1.BeaconV1, string](c, "beacons")
	return c
}

func (c *BeaconsMongoPersistence) GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error) {
	filterObj := bson.M{}
	if id, ok := filter.GetAsNullableString("id"); ok && id != "" {
		filterObj["_id"] = id
	}
	if siteId, ok := filter.GetAsNullableString("site_id"); ok && siteId != "" {
		filterObj["site_id"] = siteId
	}
	if typeId, ok := filter.GetAsNullableString("type"); ok && typeId != "" {
		filterObj["type"] = typeId
	}
	if udi, ok := filter.GetAsNullableString("udi"); ok && udi != "" {
		filterObj["udi"] = udi
	}
	if label, ok := filter.GetAsNullableString("label"); ok && label != "" {
		filterObj["label"] = label
	}
	if udis, ok := filter.GetAsObject("udis"); ok {
		var udisM bson.M
		switch _udis := udis.(type) {
		case []string:
			if len(_udis) > 0 {
				udisM = bson.M{"$in": _udis}
			}
			break
		case string:
			if _udisArr := strings.Split(_udis, ","); len(_udisArr) > 0 {
				udisM = bson.M{"$in": _udisArr}
			}
			break
		}
		if udisM != nil {
			filterObj["udi"] = udisM
		}
	}

	return c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx,
		filterObj, paging,
		nil, nil,
	)
}

func (c *BeaconsMongoPersistence) GetOneByUdi(ctx context.Context, udi string) (data1.BeaconV1, error) {

	paging := *cquery.NewPagingParams(0, 1, false)
	page, err := c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx,
		bson.M{"udi": udi}, paging,
		nil, nil,
	)
	if err != nil {
		return data1.BeaconV1{}, err
	}
	if page.HasData() {
		return page.Data[0], nil
	}
	return data1.BeaconV1{}, nil
}
```

```
persistence := NewBeaconsMongoPersistence()
// ...

persistence.Open(context.Background())

beacon := data1.BeaconV1{
	Id:     "1",
	Udi:    "0002",
	SiteId: "0001",
}

persistence.Set(context.Background(), beacon)
item, _ := persistence.GetOneByUdi(context.Background(), "0002")
persistence.Close(context.Background())
fmt.Println(item.Id)
```

```
...
# MongoDb Connection
- descriptor: "pip-services:connection:mongodb:default:1.0"
  connection:
    host: localhost
    port: 30000
...
```

```

import (
	controllers1 "github.com/pip-services-samples/service-beacons-go/controllers/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	logic "github.com/pip-services-samples/service-beacons-go/service"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type BeaconsServiceFactory struct {
	cbuild.Factory
}

func NewBeaconsServiceFactory() *BeaconsServiceFactory {
	c := &BeaconsServiceFactory{
		Factory: *cbuild.NewFactory(),
	}

	memoryPersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "memory", "*", "1.0")
	mongoPersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "mongodb", "*", "1.0")
	filePersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "file", "*", "1.0")
	serviceDescriptor := cref.NewDescriptor("beacons", "service", "default", "*", "1.0")
	httpcontrollerV1Descriptor := cref.NewDescriptor("beacons", "controller", "http", "*", "1.0")

	c.RegisterType(mongoPersistenceDescriptor, persist.NewBeaconsMongoPersistence)
	c.RegisterType(memoryPersistenceDescriptor, persist.NewBeaconsMemoryPersistence)
	c.RegisterType(filePersistenceDescriptor, persist.NewBeaconsFilePersistence)
	c.RegisterType(serviceDescriptor, logic.NewBeaconsService)
	c.RegisterType(httpcontrollerV1Descriptor, controllers1.NewBeaconsHttpControllerV1)

	return c
}
```

```
import (
	factory "github.com/pip-services-samples/service-beacons-go/build"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	rbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	cpg "github.com/pip-services4/pip-services4-go/pip-services4-postgres-go/build"
)

type BeaconsProcess struct {
	cproc.ProcessContainer
}

func NewBeaconsProcess() *BeaconsProcess {
	c := &BeaconsProcess{
		ProcessContainer: *cproc.NewProcessContainer("beacons", "Beacons microservice"),
	}

	c.AddFactory(factory.NewBeaconsServiceFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())
	c.AddFactory(cpg.NewDefaultPostgresFactory())

	return c
}

```

```
...
# MongoDb persistence
- descriptor: "beacons:persistence:mongodb:default:1.0"
  connection:
    host: localhost
    port: 30000
...
```

```
type IIdentifiable[K any] interface {
	GetId() K
}
```

```
type IdentifiableMongoDbPersistence struct {
	*MongoDbPersistence
}

func InheritIdentifiableMongoDbPersistence(overrides IMongoDbPersistenceOverrides) *IdentifiableMongoDbPersistence[T, K] {
    ...
}

func (c *IdentifiableMongoDbPersistence) Configure(ctx context.Context, config *cconf.ConfigParams) {
    ...
}

func (c *IdentifiableMongoDbPersistence) GetListByIds(ctx context.Context, ids []K) (items []T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) GetOneById(ctx context.Context, id K) (item T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) Create(ctx context.Context, item T) (result T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) Set(ctx context.Context, item T) (result T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) Update(ctx context.Context, item T) (result T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) UpdatePartially(ctx context.Context, id K, data *cdata.AnyValueMap) (item T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) DeleteById(ctx context.Context, id K) (item T, err error) {
    ...
}

func (c *IdentifiableMongoDbPersistence) DeleteByIds(ctx context.Context, ids []K) error {
    ...
}
```

```
import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cmongo "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
	"go.mongodb.org/mongo-driver/bson"
)

type BeaconsMongoDbPersistence struct {
	cmongo.IdentifiableMongoDbPersistence[data1.BeaconV1, string]
}

func NewBeaconsMongoDbPersistence() *BeaconsMongoDbPersistence {
	c := &BeaconsMongoDbPersistence{}
	c.IdentifiableMongoDbPersistence = *cmongo.InheritIdentifiableMongoDbPersistence[data1.BeaconV1, string](c, "beacons")
	return c
}

func (c *BeaconsMongoDbPersistence) composeFilter(filter *cquery.FilterParams) interface{} {
	if filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	criteria := make([]bson.M, 0, 0)

	id := filter.GetAsString("id")
	if id != "" {
		criteria = append(criteria, bson.M{"_id": id})
	}

	siteId := filter.GetAsString("site_id")
	if siteId != "" {
		criteria = append(criteria, bson.M{"site_id": siteId})
	}
	label := filter.GetAsString("label")
	if label != "" {
		criteria = append(criteria, bson.M{"label": label})
	}
	udi := filter.GetAsString("udi")
	if udi != "" {
		criteria = append(criteria, bson.M{"udi": udi})
	}

	udis := filter.GetAsString("udis")
	var arrUdis []string = make([]string, 0, 0)
	if udis != "" {
		arrUdis = strings.Split(udis, ",")
		if len(arrUdis) > 1 {
			criteria = append(criteria, bson.M{"udi": bson.D{{"$in", arrUdis}}})
		}
	}
	if len(criteria) > 0 {
		return bson.D{{"$and", criteria}}
	}
	return bson.M{}
}

func (c *BeaconsMongoDbPersistence) GetPageByFilter(ctx context.Context, correlationId string, filter *cquery.FilterParams, paging cquery.PagingParams) (page cquery.DataPage[data1.BeaconV1], err error) {
	return c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx, c.composeFilter(filter), paging, nil, nil)
}

```

```
filter := cquery.NewFilterParamsFromTuples(
	"name", "ABC",
)
result, _ := persistence.GetPageByFilter(context.Background(), filter, nil)
```

```
func (c *BeaconsMongoDbPersistence) composeFilter(filter *cquery.FilterParams) interface{} {
	if filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	criteria := make([]bson.M, 0, 0)

	id := filter.GetAsString("id")
	if id != "" {
		criteria = append(criteria, bson.M{"_id": id})
	}

	siteId := filter.GetAsString("site_id")
	if siteId != "" {
		criteria = append(criteria, bson.M{"site_id": siteId})
	}
	label := filter.GetAsString("label")
	if label != "" {
		criteria = append(criteria, bson.M{"label": label})
	}
	udi := filter.GetAsString("udi")
	if udi != "" {
		criteria = append(criteria, bson.M{"udi": udi})
	}

	udis := filter.GetAsString("udis")
	var arrUdis []string = make([]string, 0, 0)
	if udis != "" {
		arrUdis = strings.Split(udis, ",")
		if len(arrUdis) > 1 {
			criteria = append(criteria, bson.M{"udi": bson.D{{"$in", arrUdis}}})
		}
	}
	if len(criteria) > 0 {
		return bson.D{{"$and", criteria}}
	}
	return bson.M{}
}
```

```
// skip = 25, take = 50, total = False
paging := cquery.NewPagingParams(25, 50, false)
result := persistence.GetPageByFilter(context.Background(), nil, paging)
```

```
func (c *BeaconsMongoPersistence) GetOneById(ctx context.Context, id string) (item data1.BeaconV1, err error) {
	return c.IdentifiableMongoDbPersistence.GetOneById(context.Background(), id)
}
```

```
import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cmongo "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
)

type BeaconsMongoDbPersistence struct {
	cmongo.IdentifiableMongoDbPersistence[data1.BeaconV1, string]
}

func NewBeaconsMongoDbPersistence() *BeaconsMongoDbPersistence {
	c := &BeaconsMongoDbPersistence{}
	c.IdentifiableMongoDbPersistence = *cmongo.InheritIdentifiableMongoDbPersistence[data1.BeaconV1, string](c, "beacons")
	return c
}

func (c *BeaconsMongoDbPersistence) composeFilter(filter *cquery.FilterParams) interface{} {
	if filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	criteria := make([]bson.M, 0, 0)

	id := filter.GetAsString("id")
	if id != "" {
		criteria = append(criteria, bson.M{"_id": id})
	}

	siteId := filter.GetAsString("site_id")
	if siteId != "" {
		criteria = append(criteria, bson.M{"site_id": siteId})
	}
	label := filter.GetAsString("label")
	if label != "" {
		criteria = append(criteria, bson.M{"label": label})
	}
	udi := filter.GetAsString("udi")
	if udi != "" {
		criteria = append(criteria, bson.M{"udi": udi})
	}

	udis := filter.GetAsString("udis")
	var arrUdis []string = make([]string, 0, 0)
	if udis != "" {
		arrUdis = strings.Split(udis, ",")
		if len(arrUdis) > 1 {
			criteria = append(criteria, bson.M{"udi": bson.D{{"$in", arrUdis}}})
		}
	}
	if len(criteria) > 0 {
		return bson.D{{"$and", criteria}}
	}
	return bson.M{}
}

func (c *BeaconsMongoDbPersistence) GetPageByFilter(ctx context.Context, correlationId string, filter *cquery.FilterParams, paging cquery.PagingParams) (page cquery.DataPage[data1.BeaconV1], err error) {
	return c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx, c.composeFilter(filter), paging, nil, nil)
}

func (c *BeaconsMongoDbPersistence) GetOneByUdi(ctx context.Context, udi string) (result data1.BeaconV1, err error) {
	filter := bson.M{"udi": udi}
	var docPointer map[string]any
	foRes := c.Collection.FindOne(ctx, filter)

	ferr := foRes.Decode(&docPointer)
	if ferr != nil {
		if ferr == mongo.ErrNoDocuments {
			return data1.BeaconV1{}, nil
		}
		return data1.BeaconV1{}, ferr
	}

	result, err = c.ConvertToPublic(docPointer)
	if err != nil {
		return result, err
	}
	return result, nil
}
```

```
persistence := NewBeaconsMongoDbPersistence()

persistence.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", "30000",
	"connection.database", "test",
))

persistence.Open(context.Background())
beacon := &data1.BeaconV1{Id: "1", SiteId: "0001", Udi: "0002"}

persistence.Set(context.Background(), *beacon)
item, _ := persistence.GetOneByUdi(context.Background(), "0002")

fmt.Println(item.Udi) // Result: 0002

itemsPage, _ := persistence.GetPageByFilter(context.Background(), "test", cquery.NewFilterParamsFromTuples("udi", "0002"), cquery.PagingParams{})

fmt.Println(len(itemsPage.Data))   // Result: 1
fmt.Println(itemsPage.Data[0].Udi) // Result: 0002
persistence.Close(context.Background())
```
