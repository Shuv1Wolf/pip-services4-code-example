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

func (c *MyMongoDbPersistence) GetPageByFilter(ctx context.Context, filter *cquery.FilterParams, paging *cquery.PagingParams, sort *cquery.SortParams) (page *cquery.DataPage[MyData], err error) {

	return c.MongoDbPersistence.GetPageByFilter(ctx,
		c.composeFilter(filter), paging,
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

```
