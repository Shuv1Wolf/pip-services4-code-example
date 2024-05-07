```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	postgrespersist "github.com/pip-services4/pip-services4-go/pip-services4-postgres-go/persistence"
	mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
)
```

```
type MyData struct {
	Id      string `bson:"_id" json:"id"`
	Key     string `bson:"key" json:"key"`
	Content string `bson:"content" json:"content"`
}

func (d *MyData) SetId(id string) {
	d.Id = id
}

func (d MyData) GetId() string {
	return d.Id
}

func (d MyData) Clone() MyData {
	return MyData{
		Id:      d.Id,
		Key:     d.Key,
		Content: d.Content,
	}
}
```

```
type IMyDataPersistence interface {
	Set(ctx context.Context, item MyData) (result MyData, err error)

	Create(ctx context.Context, item MyData) (result MyData, err error)

	GetPageByFilter(ctx context.Context, filter cdata.FilterParams, paging cdata.PagingParams, sort cdata.SortParams) (page cdata.DataPage[MyData], err error)

	GetCountByFilter(ctx context.Context, filter cdata.FilterParams) (count int64, err error)

	GetListByFilter(ctx context.Context, filter cdata.FilterParams, sort cdata.SortParams) (items []MyData, err error)

	GetOneById(ctx context.Context, id string) (item MyData, err error)

	GetListByIds(ctx context.Context, ids []string) (items []MyData, err error)

	Update(ctx context.Context, item MyData) (result MyData, err error)

	UpdatePartially(ctx context.Context, id string, data cdata.AnyValueMap) (result MyData, err error)

	DeleteById(ctx context.Context, id string) (result MyData, err error)

	DeleteByIds(ctx context.Context, ids []string) error

	DeleteByFilter(ctx context.Context, filter cdata.FilterParams) (err error)
}
```

```
type MyIdentifiableMySqlPersistence struct {
	*mysqlpersist.IdentifiableMySqlPersistence[MyData, string]
}

func NewMyIdentifiableMySqlPersistence() *MyIdentifiableMySqlPersistence {
	c := &MyIdentifiableMySqlPersistence{}
	c.IdentifiableMySqlPersistence = mysqlpersist.InheritIdentifiableMySqlPersistence[MyData, string](c, "mydata")
	return c
}

func (c *MyIdentifiableMySqlPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE `" + c.TableName + "` (id VARCHAR(32) PRIMARY KEY, `key` VARCHAR(50), `content` TEXT)")
}

func (c *MyIdentifiableMySqlPersistence) composeFilter(filter cquery.FilterParams) string {
	key, keyOk := filter.GetAsNullableString("key")
	content, contentOk := filter.GetAsNullableString("content")

	filterObj := ""
	if keyOk && key != "" {
		filterObj += "`key`='" + key + "'"
	}
	if contentOk && content != "" {
		filterObj += "`content`='" + content + "'"
	}

	return filterObj
}

func (c *MyIdentifiableMySqlPersistence) composeSort(sort cquery.SortParams) string {
	composeSort := ""

	for _, field := range sort {
	    composeSort += field.Name
	    if field.Ascending {
	    	composeSort += " ASC"
	    } else {
	    	composeSort += " DESC"
	    }
    }

	return composeSort
}

func (c *MyIdentifiableMySqlPersistence) GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams, sort cquery.SortParams) (page cquery.DataPage[MyData], err error) {
	return c.MySqlPersistence.GetPageByFilter(ctx, c.composeFilter(filter), paging, c.composeSort(sort), "")
}

func (c *MyIdentifiableMySqlPersistence) GetListByFilter(ctx context.Context, filter cquery.FilterParams, sort cquery.SortParams) (items []MyData, err error) {

	return c.MySqlPersistence.GetListByFilter(ctx, c.composeFilter(filter), c.composeSort(sort), "")
}

func (c *MyIdentifiableMySqlPersistence) GetCountByFilter(ctx context.Context, filter cquery.FilterParams) (count int64, err error) {
	return c.MySqlPersistence.GetCountByFilter(ctx, c.composeFilter(filter))
}

func (c *MyIdentifiableMySqlPersistence) DeleteByFilter(ctx context.Context, filter cquery.FilterParams) (err error) {
	return c.MySqlPersistence.DeleteByFilter(ctx, c.composeFilter(filter))
}
```

```
type MyData struct {
	Id      string `bson:"_id" json:"id"`
	Key     string `bson:"key" json:"key"`
	Content string `bson:"content" json:"content"`
}

func (d *MyData) SetId(id string) {
	d.Id = id
}

func (d MyData) GetId() string {
	return d.Id
}

func (d MyData) Clone() MyData {
	return MyData{
		Id:      d.Id,
		Key:     d.Key,
		Content: d.Content,
	}
}

type IMyDataPersistence interface {
	Set(ctx context.Context, item MyData) (result MyData, err error)

	Create(ctx context.Context, item MyData) (result MyData, err error)

	GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams, sort cquery.SortParams) (page cquery.DataPage[MyData], err error)

	GetCountByFilter(ctx context.Context, filter cquery.FilterParams) (count int64, err error)

	GetListByFilter(ctx context.Context, filter cquery.FilterParams, sort cquery.SortParams) (items []MyData, err error)

	GetOneById(ctx context.Context, id string) (item MyData, err error)

	GetListByIds(ctx context.Context, ids []string) (items []MyData, err error)

	Update(ctx context.Context, item MyData) (result MyData, err error)

	UpdatePartially(ctx context.Context, id string, data cdata.AnyValueMap) (result MyData, err error)

	DeleteById(ctx context.Context, id string) (result MyData, err error)

	DeleteByIds(ctx context.Context, ids []string) error

	DeleteByFilter(ctx context.Context, filter cquery.FilterParams) (err error)
}
type MyPostgresPersistence struct {
	*postgrespersist.IdentifiablePostgresPersistence[MyData, string]
}

func NewMyPostgresPersistence() *MyPostgresPersistence {
	c := &MyPostgresPersistence{}
	c.IdentifiablePostgresPersistence = postgrespersist.InheritIdentifiablePostgresPersistence[MyData, string](c, "mydata")
	return c
}

func (c *MyPostgresPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE " + c.QuotedTableName() + " (\"id\" TEXT PRIMARY KEY, \"key\" TEXT, \"content\" TEXT)")
}

func (c *MyPostgresPersistence) composeFilter(filter cquery.FilterParams) string {
	key, keyOk := filter.GetAsNullableString("key")
	content, contentOk := filter.GetAsNullableString("content")

	filterObj := ""
	if keyOk && key != "" {
		filterObj += "key='" + key + "'"
	}
	if contentOk && content != "" {
		filterObj += "content='" + content + "'"
	}

	return filterObj
}

func (c *MyPostgresPersistence) composeSort(sort cquery.SortParams) string {
	composeSort := ""

	for _, field := range sort {
		composeSort += field.Name
		if field.Ascending {
			composeSort += " ASC"
		} else {
			composeSort += " DESC"
		}
	}

	return composeSort
}

func (c *MyPostgresPersistence) GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams, sort cquery.SortParams) (page cquery.DataPage[MyData], err error) {
	return c.PostgresPersistence.GetPageByFilter(ctx, c.composeFilter(filter), paging, c.composeSort(sort), "")
}

func (c *MyPostgresPersistence) GetListByFilter(ctx context.Context, filter cquery.FilterParams, sort cquery.SortParams) (items []MyData, err error) {

	return c.PostgresPersistence.GetListByFilter(ctx, c.composeFilter(filter), c.composeSort(sort), "")
}

func (c *MyPostgresPersistence) GetCountByFilter(ctx context.Context, filter cquery.FilterParams) (count int64, err error) {
	return c.PostgresPersistence.GetCountByFilter(ctx, c.composeFilter(filter))
}

func (c *MyPostgresPersistence) DeleteByFilter(ctx context.Context, filter cquery.FilterParams) (err error) {
	return c.PostgresPersistence.DeleteByFilter(ctx, c.composeFilter(filter))
}
```

```
host := "localhost"
port := "3306"
db_name := "pip"
user := "root"
password := ""
```

```
database1 := NewMyMySqlPersistence()
database1.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", host,
	"connection.port", port,
	"connection.database", db_name,
	"credential.username", user,
	"credential.password", password,
))
```

```
err := database1.Open(context.Background())
```

```
host := "localhost"
port := "5432"
db_name := "test"
user := "postgres"
password := "postgres"
```

```
database2 := NewMyPostgresPersistence()
database2.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", host,
	"connection.port", port,
	"connection.database", db_name,
	"credential.username", user,
	"credential.password", password,
))
```

```

err = database2.Open(context.Background())
```

```
var persistence IMyDataPersistence
```

```
persistence = database1
```

```
persistence = database2
```

```
for i := 0; i < 20; i++ {
	index := strconv.Itoa(i)
	data := MyData{Id: index, Key: "key " + index, Content: "content " + index}
	res, err := persistence.Create(context.Background(), data)
	if err != nil {
		panic(err)
	}
	fmt.Println(res)
}
```

```
ids := []string{"3", "4", "5"}
myDataList, err := persistence.GetListByIds(context.Background(), ids)
```

```
result, err := persistence.GetPageByFilter(context.Background(), *cquery.NewFilterParamsFromTuples("key", "key 8"), *cquery.NewEmptyPagingParams(), *cquery.NewEmptySortParams())
```

```
result.Data[0].Id;       // Returns '8'
result.Data[0].Key;      // Returns 'key 8'
result.Data[0].Content;  // Returns 'content 8'
```

```
newValue := MyData{Id: "1", Key: "key 1", Content: "Updated content 1"}
res, err := persistence.Update(context.Background(), newValue)
```

```
ids = []string{"0", "1"}
err = persistence.DeleteByIds(context.Background(), ids)
```

```
err = persistence.DeleteByFilter(context.Background(), *cquery.NewFilterParamsFromTuples("key", "key 7"))
```

```
// Step 1:  we extract the data from the MySQL database
persistence = database1
ids3 := []string{"1", "2", "3", "4", "5", "6", "7", "8", "9", "10"}
myDataList, err = persistence.GetListByIds(context.Background(), ids3)
if err != nil {
	panic(err)
}
// Step 2: we insert the data into the mydata table in the PostgreSQL database.
persistence = database2
for _, item := range myDataList {
	result, err := persistence.Create(context.Background(), item)
	if err != nil {
		panic(err)
	}
	fmt.Println(result)
}
```
