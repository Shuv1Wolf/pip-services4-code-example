```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-sqlserver-go@latest
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
data1 := MyData{Id: "1", Key: "key 1", Content: "content 1"}
data2 := MyData{Id: "2", Key: "key 2", Content: "content 2"}
data3 := MyData{Id: "3", Key: "key 3", Content: "content 3"}
```

```
import (
	sqlsrvpersist "github.com/pip-services4/pip-services4-go/pip-services4-sqlserver-go/persistence"
)
```

```
import (
	"context"

	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	sqlsrvpersist "github.com/pip-services4/pip-services4-go/pip-services4-sqlserver-go/persistence"
)

type MySqlServerPersistence struct {
	*sqlsrvpersist.SqlServerPersistence[MyData]
}

func NewMySqlServerPersistence() *MySqlServerPersistence {
	c := &MySqlServerPersistence{}
	c.SqlServerPersistence = sqlsrvpersist.InheritSqlServerPersistence[MyData](c, "mydata")
	return c
}

func (c *MySqlServerPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE [" + c.TableName + "] ([id] VARCHAR(32) PRIMARY KEY, [key] VARCHAR(50), [content] VARCHAR(MAX))")
	c.EnsureIndex(c.SqlServerPersistence.TableName+"_key", map[string]string{"key": "1"}, map[string]string{"unique": "true"})
}

func (c *MySqlServerPersistence) GetPageByFilter(ctx context.Context, filter string,
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.SqlServerPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MySqlServerPersistence) GetListByFilter(ctx context.Context, filter string,
	sort string, selection string) (items []MyData, err error) {

	return c.SqlServerPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MySqlServerPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.SqlServerPersistence.GetCountByFilter(ctx, filter)
}

func (c *MySqlServerPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.SqlServerPersistence.GetOneRandom(ctx, filter)
}
```

```
import cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

persistence := NewMySqlServerPersistence()
persistence.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 1433,
	"connection.database", "pip",
	"credential.username", "user",
	"credential.password", "password",
))
```

```
err := persistence.Open(context.Background())
```

```
result, err := persistence.Create(context.Background(), data1)
```

```
result.Id;       // Returns '1'
result.Key;      // Returns 'key 1'
result.Content;  // Returns 'content 1'

```

```
result, err := persistence.GetOneRandom(context.Background(), "[key]='key 3'")
```

```
result.Id;       // Returns '3'
result.Key;      // Returns 'key 3'
result.Content;  // Returns 'content 3'

```

```
result, err := persistence.GetListByFilter(context.Background(), "[key]='key 3'", "", "")
```

```
result[0].Id;       // Returns '3'
result[0].Key;      // Returns 'key 3'
result[0].Content;  // Returns 'content 3'

```

```
page, err := persistence.GetPageByFilter(context.Background(), "[key]='key 3'", *cquery.NewEmptyPagingParams(), "", "")
```

```
page.Data[0].Id      // Returns '1'
page.Data[0].Key     // Returns 'key 1'
page.Data[0].Content // Returns 'content 5'
```

```
count, err := persistence.GetCountByFilter(context.Background(), "[key]='key 3'") // Returns 1
```

```
err = persistence.DeleteByFilter(context.Background(), "[key]='key 3'")
```

```
import (
	sqlsrvpersist "github.com/pip-services4/pip-services4-go/pip-services4-sqlserver-go/persistence"
)
```

```
type MySqlServerPersistence struct {
	*sqlsrvpersist.IdentifiableSqlServerPersistence[MyData, string]
}

func NewMySqlServerPersistence() *MySqlServerPersistence {
	c := &MySqlServerPersistence{}
	c.IdentifiableSqlServerPersistence = sqlsrvpersist.InheritIdentifiableSqlServerPersistence[MyData, string](c, "mydata2")
	return c
}

func (c *MySqlServerPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureSchema("CREATE TABLE [" + c.TableName + "] ([id] VARCHAR(32) PRIMARY KEY, [key] VARCHAR(50), [content] VARCHAR(MAX))")
	c.EnsureIndex(c.SqlServerPersistence.TableName+"_key", map[string]string{"key": "1"}, map[string]string{"unique": "true"})
}

func (c *MySqlServerPersistence) GetPageByFilter(ctx context.Context, filter string,
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.SqlServerPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MySqlServerPersistence) GetListByFilter(ctx context.Context, filter string,
	sort string, selection string) (items []MyData, err error) {

	return c.SqlServerPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MySqlServerPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.SqlServerPersistence.GetCountByFilter(ctx, filter)
}

func (c *MySqlServerPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.SqlServerPersistence.GetOneRandom(ctx, filter)
}
```

```
import cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

persistence := NewMySqlServerPersistence()
persistence.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 1433,
	"connection.database", "pip",
	"credential.username", "user",
	"credential.password", "password",
))
```

```
err := persistence.Open(context.Background())
```

```
res, err := persistence.Create(context.Background(), data1)
```

```
res.Id       // Returns '1'
res.Key      // Returns 'key 1'
res.Content  // Returns 'content 1'

```

```
result, err := persistence.GetOneById(context.Background(), "3")
```

```
result.Id       // Returns '3'
result.Key      // Returns 'key 3'
result.Content  // Returns 'content 3'

```

```
ids := []string{"1", "2", "3"}
items, err := persistence.GetListByIds(context.Background(), ids)
```

```

for _, item := range items {
	fmt.Println(item.Id, item.Key, item.Content)
}

// Prints
// 1 new key 1 content 1
// 3 key 3 content 3
// 2 key 2 content 2
```

```
result, err := persistence.Update(context.Background(), MyData{Id: "2", Key: "key 2.1", Content: "new content 2"})
```

```
result.Id       // Returns '2'
result.Key      // Returns 'key 2.1'
result.Content  // Returns 'new content 2'

```

```
result, err := persistence.UpdatePartially(context.Background(), "1", *cdata.NewAnyValueMapFromTuples("key", "new key 1.1"))
```

```
result.Id       // Returns '1'
result.Key      // Returns 'new key 1.1'
result.Content  // Returns 'content 1'

```

```
res, err = persistence.DeleteById(context.Background(), "1")
```

```
result.Id       // Returns '1'
result.Key      // Returns 'key 1'
result.Content  // Returns 'content 1'

```

```
ids := []string{"1", "2", "3"}
err := persistence.DeleteByIds(context.Background(), ids)

```

```
import (
	sqlsrvpersist "github.com/pip-services4/pip-services4-go/pip-services4-sqlserver-go/persistence"
)
```

```
type MySqlServerPersistence struct {
	*sqlsrvpersist.IdentifiableJsonSqlServerPersistence[MyData, string]
}

func NewMySqlServerPersistence() *MySqlServerPersistence {
	c := &MySqlServerPersistence{}
	c.IdentifiableJsonSqlServerPersistence = sqlsrvpersist.InheritIdentifiableJsonSqlServerPersistence[MyData, string](c, "mydata_json")
	return c
}

func (c *MySqlServerPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureTable("", "")
	c.EnsureSchema("ALTER TABLE [" + c.TableName + "] ADD [data_key] AS JSON_VALUE([data],'$.key')")
	c.EnsureIndex(c.SqlServerPersistence.TableName+"_key", map[string]string{"key": "1"}, map[string]string{"unique": "true"})
}

func (c *MySqlServerPersistence) GetPageByFilter(ctx context.Context, filter string,
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.SqlServerPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MySqlServerPersistence) GetListByFilter(ctx context.Context, filter string,
	sort string, selection string) (items []MyData, err error) {

	return c.SqlServerPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MySqlServerPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.SqlServerPersistence.GetCountByFilter(ctx, filter)
}

func (c *MySqlServerPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.SqlServerPersistence.GetOneRandom(ctx, filter)
}

```

```
result, err := persistence.GetOneRandom(context.Background(), "[key]='key 3'")

```

```
result.Id;      // Returns '3'
result.Key;     // Returns 'key 3'
result.Content; // Returns 'content 3'

```
