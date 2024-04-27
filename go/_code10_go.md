### Connection Management

```
# Database connection
descriptor: pip-services:connection:mongodb:default:3.0
connection:
  host: ${{MONGO_HOST}}{$unless MONGO_HOST}localhost{$unless}
  port: ${{MONGO_PORT}}{$unless MONGO_PORT}27017{$unless}
credentials:
  host: ${{MONGO_USER}}{$unless MONGO_USER}mongo{$unless}
  port: ${{MONGO_PASS}}{$unless MONGO_PASS}pwd123{$unless}
options:
  max_pool_size: 10
  keep_alive: true

```

```
# Persistence with default connection
descriptor: myservice:mypersistence:mongodb:persist1:1.0

# Persistence linked to specific connection
descriptor: myservice:mypersistence:mongodb:persist2:1.0
references:
  connection: pip-services:connection:mongodb:conn1:3.0

```

### Generic Persistence

```
import (
	"context"
	"strings"

	persist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
	"go.mongodb.org/mongo-driver/bson"
)

type MyObject struct {
	Key  string `bson:"key" json:"key"`
	Name string `bson:"name" json:"name"`
}

type MyMongoDbPersistence struct {
	*persist.MongoDbPersistence[MyObject]
}

func NewMyMongoDbPersistence() *MyMongoDbPersistence {
	c := &MyMongoDbPersistence{}
	c.MongoDbPersistence = persist.InheritMongoDbPersistence[MyObject](c, "mycollection")
	return c
}

func (c *MyMongoDbPersistence) GetByName(ctx context.Context, name string) (result MyObject, err error) {

	filterObj := bson.M{"name": name}

	items, err := c.MongoDbPersistence.GetListByFilter(ctx, filterObj, nil, nil)
	if err != nil {
		return result, err
	}

	if len(items) > 0 {
		return items[0], nil
	} else {
		return result, nil
	}
}

func (c *MyMongoDbPersistence) CreateDefault(ctx context.Context, correlationId string,
	name string) (result MyObject, err error) {

	if name == "" {
		name = "unknown"
	}

	key := strings.ReplaceAll(strings.ToLower(name), " #$%^&", "_")
	item := MyObject{Key: key, Name: name}

	newItem, err := c.Overrides.ConvertFromPublic(item)
	if err != nil {
		return result, err
	}
	insRes, err := c.Collection.InsertOne(ctx, newItem)
	if err != nil {
		return result, err
	}

	result, err = c.Overrides.ConvertToPublic(newItem)
	if err != nil {
		return result, err
	}
	c.Logger.Trace(ctx, correlationId, "Created in %s with id = %s", c.Collection, insRes.InsertedID)
	return result, nil
}

func (c *MyMongoDbPersistence) DeleteByName(ctx context.Context, name string) error {
	filterObj := bson.M{"name": name}
	return c.DeleteByFilter(ctx, filterObj)
}

```

### Identifiable Persistence

```
type MyIdentifiableObject struct {
	Id    string `bson:"id" json:"id"`
	Name  string `bson:"name" json:"name"`
	Value string `bson:"value" json:"value"`
}

func (c *MyIdentifiableObject) GetId() string {
	return c.Id
}
```

```
import (
	"context"
	"strings"

	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	persist "github.com/pip-services4/pip-services4-go/pip-services4-postgres-go/persistence"
)

type MyIdentifiableObject struct {
	Id    string `bson:"id" json:"id"`
	Name  string `bson:"name" json:"name"`
	Value string `bson:"value" json:"value"`
}

func (c *MyIdentifiableObject) GetId() string {
	return c.Id
}

type MyIdentifiablePersistence interface {
	GetPageByFilter(correlationId string, filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[MyIdentifiableObject], error)
	Create(correlationId string, item MyIdentifiableObject) (MyIdentifiableObject, error)
	GetOneById(correlationId string, id string) (MyIdentifiableObject, error)
	DeleteById(correlationId string, id string) (MyIdentifiableObject, error)
}

type MyIdentifiablePostgreSqlPersistence struct {
	*persist.IdentifiablePostgresPersistence[MyIdentifiableObject, string]
}

func NewMyIdentifiablePostgreSqlPersistence() *MyIdentifiablePostgreSqlPersistence {
	c := &MyIdentifiablePostgreSqlPersistence{}
	c.IdentifiablePostgresPersistence = persist.InheritIdentifiablePostgresPersistence[MyIdentifiableObject, string](c, "mycollection")
	return c
}

func (c *MyIdentifiablePostgreSqlPersistence) composeFilter(filter cquery.FilterParams) string {
	criteria := make([]string, 0)

	if id, ok := filter.GetAsNullableString("id"); ok && id != "" {
		criteria = append(criteria, "id='"+id+"'")
	}

	if name, ok := filter.GetAsNullableString("name"); ok && name != "" {
		criteria = append(criteria, "name='"+name+"'")
	}

	if len(criteria) > 0 {
		return strings.Join(criteria, " AND ")
	} else {
		return ""
	}
}

func (c *MyIdentifiablePostgreSqlPersistence) GetPageByFilter(ctx context.Context,
	filter cquery.FilterParams, paging cquery.PagingParams) (page cquery.DataPage[MyIdentifiableObject], err error) {

	return c.IdentifiablePostgresPersistence.GetPageByFilter(ctx,
		c.composeFilter(filter), paging,
		"", "",
	)
}
```

```
CREATE TABLE MyIdentifiableJsonObject (
  id VARCHAR(32) PRIMARY KEY,
  data JSON
);
```

### Custom Persistence

```
import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type MyCustomPersistence struct {
	// Custom implementation using any persistence framework
}

func NewMyCustomPersistence() *MyCustomPersistence {
	return &MyCustomPersistence{}
}

type MyCustomPersistenceWrapper struct {
	config      *cconf.ConfigParams
	persistence *MyCustomPersistence
}

func NewMyCustomPersistenceWrapper() *MyCustomPersistenceWrapper {
	return &MyCustomPersistenceWrapper{
		config: cconf.NewConfigParams(map[string]string{}),
	}
}

func (c *MyCustomPersistenceWrapper) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// Store config parameters
	if config != nil {
		c.config = config
	}
}

func (c *MyCustomPersistenceWrapper) SetReferences(ctx context.Context, references cref.IReferences) {
	// Retrieve whatever references you may need
}

func (c *MyCustomPersistenceWrapper) IsOpen() bool {
	return c.persistence != nil
}

func (c *MyCustomPersistenceWrapper) Open(ctx context.Context, correlationId string) (err error) {
	if c.persistence != nil {
		return nil
	}

	// Create custom persistence
	c.persistence = NewMyCustomPersistence()

	// Configure custom persistence
	// ...

	// Open and connect to the database
	c.persistence.Connect()
}

func (c *MyCustomPersistenceWrapper) Close(ctx context.Context, correlationId string) (err error) {
	if c.persistence == nil {
		return
	}

	// Disconnect from the database and close
	c.persistence.Disconnect()
	c.persistence = nil
}

```
