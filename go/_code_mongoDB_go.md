```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go@latest
```

```
type MyData struct {
	Id      string `bson:"_id" json:"id"`
	Key     string `bson:"key" json:"key"`
	Content string `bson:"content" json:"content"`
}

type MyDataPage struct {
	Total *int64   `bson:"total" json:"total"`
	Data  []MyData `bson:"data" json:"data"`
}
```

```
data1 := MyData{Id: "1", Key: "key 1", Content: "content 1"}
data2 := MyData{Id: "2", Key: "key 2", Content: "content 2"}
data3 := MyData{Id: "3", Key: "key 3", Content: "content 3"}

```

```
import (
	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)
```

```
type MyMongoDbPersistence struct {
	*mpersist.MongoDbPersistence[MyData]
}

func NewMyMongoDbPersistence() *MyMongoDbPersistence {
	c := &MyMongoDbPersistence{}
	c.MongoDbPersistence = mpersist.InheritMongoDbPersistence(c, "mydata")
	return c
}

persistence := NewMyMongoDbPersistence()
config := conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 27017,
	"connection.database", "pipdatabase",
)
persistence.Configure(context.Background(), config)

err := persistence.Open(context.Background())
```

```
err := persistence.Open(context.Background())
```

```
item, err := persistence.Create(context.Background(), data1)
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
import (
    cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

item, err = persistence.GetOneRandom(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 3"))
```

```

result.Id      // Returns '3'
result.Key     // Returns 'key 3'
result.Content // Returns 'content 3'
```

```

func (c *MyMongoDbPersistence) GetListByFilter(ctx context.Context, filter *cquery.FilterParams, sort *cquery.SortParams) (items []MyData, err error) {
	return c.MongoDbPersistence.GetListByFilter(ctx, c.composeFilter(filter), c.composeSort(sort), nil)
}
```

```
list, err := persistence.GetListByFilter(context.Background(), cdata.NewFilterParamsFromTuples("key", "key 3"), nil)
```

```
result[0].Id      // Returns '3'
result[0].Key     // Returns 'key 3'
result[0].Content // Returns 'content 3'
```

```
func (c *MyMongoDbPersistence) composeFilter(filter *cquery.FilterParams) bson.M {
	if &filter == nil || filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	key, _ := filter.GetAsNullableString("key")
	var filterObj bson.M
	if key != "" {
		filterObj = bson.M{"key": key}
	} else {
		filterObj = bson.M{}
	}

	return filterObj
}

func (c *MyMongoDbPersistence) composeSort(sort *cquery.SortParams) bson.M {
	if &sort == nil || sort == nil {
		sort = cquery.NewEmptySortParams()
	}

	sortObj := bson.M{}

	for _, field := range *sort {
		if field.Ascending {
			sortObj[field.Name] = 1
		} else {
			sortObj[field.Name] = -1
		}

	}

	return sortObj
}
```

```
func (c *MyMongoDbPersistence) GetPageByFilter(ctx context,Context, correlationId string, filter *cdata.FilterParams, paging *cdata.PagingParams, sort *cdata.SortParams) (page *DataPage[MyData], err error) {

	return c.MongoDbPersistence.GetPageByFilter(ctx, correlationId,
		c.composeFilter(filter), paging,
		c.composeSort(sort), nil)
}
```

```
page, err := persistence.GetPageByFilter(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 3"), nil, nil)
```

```
result.Data[0].Id       // Returns '3'
result.Data[0].Key      // Returns 'key 3'
result.Data[0].Content  // Returns 'content 3'
```

```
func (c *MyMongoDbPersistence) GetCountByFilter(ctx context.Context, filter *cquery.FilterParams) (count int64, err error) {
	return c.MongoDbPersistence.GetCountByFilter(ctx, c.composeFilter(filter))
}
```

```
count, err := persistence.GetCountByFilter(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 3")) // Returns 1
```

```
err = persistence.DeleteByFilter(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 3"))
```

```
import (
	"context"

	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	"go.mongodb.org/mongo-driver/bson"

	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)

type MyMongoDbPersistence struct {
	*mpersist.MongoDbPersistence[MyData]
}

func NewMyMongoDbPersistence() *MyMongoDbPersistence {
	c := &MyMongoDbPersistence{}
	c.MongoDbPersistence = mpersist.InheritMongoDbPersistence(c, "mydata")
	return c
}

func (c *MyMongoDbPersistence) composeFilter(filter *cquery.FilterParams) bson.M {
	if &filter == nil || filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	key, _ := filter.GetAsNullableString("key")
	var filterObj bson.M
	if key != "" {
		filterObj = bson.M{"key": key}
	} else {
		filterObj = bson.M{}
	}

	return filterObj
}

func (c *MyMongoDbPersistence) composeSort(sort *cquery.SortParams) bson.M {
	if &sort == nil || sort == nil {
		sort = cquery.NewEmptySortParams()
	}

	sortObj := bson.M{}

	for _, field := range *sort {
		if field.Ascending {
			sortObj[field.Name] = 1
		} else {
			sortObj[field.Name] = -1
		}

	}

	return sortObj
}

func (c *MyMongoDbPersistence) GetListByFilter(ctx context.Context, filter *cquery.FilterParams, sort *cquery.SortParams) (items []MyData, err error) {
	return c.MongoDbPersistence.GetListByFilter(ctx, c.composeFilter(filter), c.composeSort(sort), nil)
}

func (c *MyMongoDbPersistence) GetPageByFilter(ctx context.Context, filter *cquery.FilterParams, paging *cquery.PagingParams, sort *cquery.SortParams) (page cquery.DataPage[MyData], err error) {

	return c.MongoDbPersistence.GetPageByFilter(ctx,
		c.composeFilter(filter), *paging,
		c.composeSort(sort), nil)
}

func (c *MyMongoDbPersistence) GetCountByFilter(ctx context.Context, filter *cquery.FilterParams) (count int64, err error) {
	return c.MongoDbPersistence.GetCountByFilter(ctx, c.composeFilter(filter))
}

func (c *MyMongoDbPersistence) DeleteByFilter(ctx context.Context, filter *cquery.FilterParams) error {
	return c.MongoDbPersistence.DeleteByFilter(ctx, c.composeFilter(filter))
}
```

```
import (
	"context"
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	mngoptions "go.mongodb.org/mongo-driver/mongo/options"

	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)

type MyMongoDbPersistence struct {
	*mpersist.MongoDbPersistence[MyData]
}

func NewMyMongoDbPersistence() *MyMongoDbPersistence {
	c := &MyMongoDbPersistence{}
	c.MongoDbPersistence = mpersist.InheritMongoDbPersistence(c, "mydata")
	return c
}

func (c *MyMongoDbPersistence) composeFilter(filter *cquery.FilterParams) bson.M {
	if &filter == nil || filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	key, _ := filter.GetAsNullableString("key")
	var filterObj bson.M
	if key != "" {
		filterObj = bson.M{"key": key}
	} else {
		filterObj = bson.M{}
	}

	return filterObj
}

func (c *MyMongoDbPersistence) composeSort(sort *cquery.SortParams) bson.M {
	if &sort == nil || sort == nil {
		sort = cquery.NewEmptySortParams()
	}

	sortObj := bson.M{}

	for _, field := range *sort {
		if field.Ascending {
			sortObj[field.Name] = 1
		} else {
			sortObj[field.Name] = -1
		}

	}

	return sortObj
}

func (c *MyMongoDbPersistence) Update(ctx context.Context, correlationId string, item MyData) (result MyData, err error) {
	newItem, err := c.ConvertFromPublic(item)
	id := newItem["_id"]
	filter := bson.M{"_id": id}
	update := bson.D{{"$set", newItem}}
	var options mngoptions.FindOneAndUpdateOptions
	retDoc := mngoptions.After
	options.ReturnDocument = &retDoc
	fuRes := c.Collection.FindOneAndUpdate(ctx, filter, update, &options)
	if fuRes.Err() != nil {
		return result, fuRes.Err()
	}
	c.Logger.Trace(context.Background(), "Updated in %s with id = %s", c.CollectionName, id)
	var docPointer *MyData
	err = fuRes.Decode(&docPointer)
	if err != nil {
		if err == mongo.ErrNoDocuments {
			return result, nil
		}
		return result, err
	}

	result, err = c.Overrides.ConvertToPublic(docPointer)
	if err != nil {
		return MyData{}, nil
	}
	return result, nil
}

func (c *MyMongoDbPersistence) GetListByFilter(ctx context.Context, filter *cquery.FilterParams, sort *cquery.SortParams) (items []MyData, err error) {
	return c.MongoDbPersistence.GetListByFilter(ctx, c.composeFilter(filter), c.composeSort(sort), nil)
}

func (c *MyMongoDbPersistence) DeleteByFilter(ctx context.Context, filter *cquery.FilterParams) error {
	return c.MongoDbPersistence.DeleteByFilter(ctx, c.composeFilter(filter))
}

type MyData struct {
	Id      string `bson:"_id" json:"id"`
	Key     string `bson:"key" json:"key"`
	Content string `bson:"content" json:"content"`
}

type MyDataPage struct {
	Total *int64   `bson:"total" json:"total"`
	Data  []MyData `bson:"data" json:"data"`
}

func PrintResult(operationName string, res MyData) {
	fmt.Println("==================== " + operationName + " ====================")
	fmt.Println("MyData with Id: " + res.Id)
	fmt.Println("MyData with Key: " + res.Key)
	fmt.Println("MyData with Content: " + res.Content)
}

func main() {
	data1 := MyData{Id: "1", Key: "key 1", Content: "content 1"}

	persistence := NewMyMongoDbPersistence()
	config := conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 27017,
		"connection.database", "pipdatabase",
	)
	persistence.Configure(context.Background(), config)

	_ = persistence.Open(context.Background())
	_ = persistence.Clear(context.Background())

	// 1 - Create
	result, _ := persistence.Create(context.Background(), data1)
	PrintResult("Create", result)

	// 2 - Retrieve
	items, _ := persistence.GetListByFilter(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 1"), nil)
	PrintResult("Get by id", items[0])

	// 3 - Update
	items[0].Content = "new content 2"
	items[0].Key = "key 2"

	update, _ := persistence.Update(context.Background(), "123", items[0])
	PrintResult("Update", update)

	// 4 - Delete
	_ = persistence.DeleteByFilter(context.Background(), cquery.NewFilterParamsFromTuples("key", "key 1"))

	_ = persistence.Close(context.Background())
}

```

```
import (
	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)

type MyIdentifiableMongoDbPersistence struct {
	mpersist.IdentifiableMongoDbPersistence[MyData, string]
}

func NewMyIdentifiableMongoDbPersistencee() *MyIdentifiableMongoDbPersistence {
	c := &MyIdentifiableMongoDbPersistence{}
	c.IdentifiableMongoDbPersistence = *mpersist.InheritIdentifiableMongoDbPersistence[MyData, string](c, "mydata")
	return c
}


persistence := NewMyIdentifiableMongoDbPersistencee()
config := conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 27017,
	"connection.database", "pipdatabase",
)
persistence.Configure(context.Background(), config)

```

```
_ = persistence.Open(context.Background())
```

```
_ = persistence.Clear(context.Background())
```

```
result, _ := persistence.Create(context.Background(), data1)
```

```
aresult.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
data1 = MyData{Id: "1", Key: "key 1", Content: "new content 1"}
result, _ = persistence.Set(context.Background(), data1)
```

```

result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'new content 1'
```

```
result, _ = persistence.GetOneById(context.Background(), "1")
```

```

result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
list, _ := persistence.GetListByIds(context.Background(), []string{"1", "2"})
```

```
list[0].Id      // Returns '1'
list[0].Key     // Returns 'key 1'
list[0].Content // Returns 'content 1'
list[1].Id      // Returns '2'
list[1].Key     // Returns 'key 2'
list[1].Content // Returns 'content 2'
```

```
updated, _ := persistence.Update(context.Background(), MyData{Id: "2", Key: "key 2", Content: "new content 2"})
```

```

result.Id      // Returns '2'
result.Key     // Returns 'key 2'
result.Content // Returns 'new content 2'
```

```
import cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"

updated, _ = persistence.UpdatePartially(context.Background(), "2", *cdata.NewAnyValueMapFromTuples(
	"content", "new content 2 - partially updated",
))
```

```
result.Id      // Returns '2'
result.Key     // Returns 'key 2'
result.Content // Returns 'new content 2 - partially updated'
```

````
deleted, _ := persistence.DeleteById(context.Background(), "1")
````

```
result.Id       // Returns '1'
result.Key      // Returns 'key 1'
result.Content  // Returns 'content 1'
```

```
_ = persistence.DeleteByIds(context.Background(), []string{"1", "2"})
```

```
import (
	"context"
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	mpersist "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
)

type MyData struct {
	Id      string `bson:"_id" json:"id"`
	Key     string `bson:"key" json:"key"`
	Content string `bson:"content" json:"content"`
}

type MyIdentifiableMongoDbPersistence struct {
	mpersist.IdentifiableMongoDbPersistence[MyData, string]
}

func NewMyIdentifiableMongoDbPersistencee() *MyIdentifiableMongoDbPersistence {
	c := &MyIdentifiableMongoDbPersistence{}
	c.IdentifiableMongoDbPersistence = *mpersist.InheritIdentifiableMongoDbPersistence[MyData, string](c, "mydata")
	return c
}

func PrintResult(operationName string, res MyData) {
	fmt.Println("==================== " + operationName + " ====================")
	fmt.Println("MyData with Id: " + res.Id)
	fmt.Println("MyData with Key: " + res.Key)
	fmt.Println("MyData with Content: " + res.Content)
}

func main() {
	data1 := MyData{Id: "1", Key: "key 1", Content: "content 1"}

	persistence := NewMyIdentifiableMongoDbPersistencee()
	config := conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 27017,
		"connection.database", "pipdatabase",
	)
	persistence.Configure(context.Background(), config)

	_ = persistence.Open(context.Background())
	_ = persistence.Clear(context.Background())

	// CRUD
	// 1 - Create
	result, _ := persistence.Create(context.Background(), data1)
	PrintResult("Create", result)

	// 2 - Retrieve
	item, _ := persistence.GetOneById(context.Background(), "1")
	PrintResult("Get by id", item)

	// 3 - Update
	update, _ := persistence.Update(context.Background(), MyData{Id: "1", Key: "key 2", Content: "new content 2"})
	PrintResult("Update", update)

	// 4 - Delete
	delete, _ := persistence.DeleteById(context.Background(), "1")
	PrintResult("Delete by id", delete)

	_ = persistence.Close(context.Background())
}
```
