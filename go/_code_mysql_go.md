```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence@latest
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
      mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
)
```

```
import (
	"context"

	mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)


type MyMySqlPersistence struct {
	*mysqlpersist.MySqlPersistence[MyData]
}

func NewMyMySqlPersistence() *MyMySqlPersistence {
	c := &MyMySqlPersistence{}
	c.MySqlPersistence = mysqlpersist.InheritMySqlPersistence[MyData](c, "mydata")
	return c
}

func (c *MyMySqlPersistence) DefineSchema() {
	c.ClearSchema()
	c.MySqlPersistence.DefineSchema()
	// Row name must be in double quotes for properly case!!!
	c.EnsureSchema("CREATE TABLE `" + c.TableName + "` (id VARCHAR(32) PRIMARY KEY, `key` VARCHAR(50), `content` TEXT)")
	c.EnsureIndex(c.TableName+"_key", map[string]string{"key": "1"}, map[string]string{"unique": "true"})
}

func (c *MyMySqlPersistence) GetPageByFilter(ctx context.Context, filter string, 
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.MySqlPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MyMySqlPersistence) GetListByFilter(ctx context.Context, filter string, 
	sort string, selection string) (items []MyData, err error) {

	return c.MySqlPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MyMySqlPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.MySqlPersistence.GetCountByFilter(ctx, filter)
}

func (c *MyMySqlPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.MySqlPersistence.GetOneRandom(ctx, filter)
}

func (c *MyMySqlPersistence) DeleteByFilter(ctx context.Context, filter string) (err error) {
	return c.MySqlPersistence.DeleteByFilter(ctx, filter)
}
```

```
import cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

persistence := NewMyMySqlPersistence()
persistence.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 3306,
	"credential.username", "root",
	"credential.password", "",
	"connection.database", "pip",
))
```

```
err := persistence.Open(context.Background())
```

```
res, err := persistence.Create(context.Background(), data1)
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
result, err := persistence.GetOneRandom(context.Background(), "`key`='key 1'")
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
itemsList, err := persistence.GetListByFilter(context.Background(), "`key`='key 1'", "", "")
```

```
itemsList[0].Id      // Returns '1'
itemsList[0].Key     // Returns 'key 1'
itemsList[0].Content // Returns 'content 1'
```

```
page, err := persistence.GetPageByFilter(context.Background(), "`key`='key 1'", *cquery.NewEmptyPagingParams(), "", "")
```

```
page.Data[0].Id      // Returns '1'
page.Data[0].Key     // Returns 'key 1'
page.Data[0].Content // Returns 'content 1'
```

```
count, err := persistence.GetCountByFilter(context.Background(), "`key`='key 1'")
```

```
err = persistence.DeleteByFilter(context.Background(), "`key`='key 1'")
```

```
import (
      mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
)
```

```
type MyMySqlPersistence struct {
	*mysqlpersist.IdentifiableMySqlPersistence[MyData, string]
}

func NewMyMySqlPersistence() *MyMySqlPersistence {
	c := &MyMySqlPersistence{}
	c.IdentifiableMySqlPersistence = mysqlpersist.InheritIdentifiableMySqlPersistence[MyData, string](c, "mydata")
	return c
}

func (c *MyMySqlPersistence) DefineSchema() {
	c.ClearSchema()
	c.MySqlPersistence.DefineSchema()
	// Row name must be in double quotes for properly case!!!
	c.EnsureSchema("CREATE TABLE `" + c.TableName + "` (id VARCHAR(32) PRIMARY KEY, `key` VARCHAR(50), `content` TEXT)")
	c.EnsureIndex(c.TableName+"_key", map[string]string{"key": "1"}, map[string]string{"unique": "true"})
}

func (c *MyMySqlPersistence) GetPageByFilter(ctx context.Context, filter string,
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.MySqlPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MyMySqlPersistence) GetListByFilter(ctx context.Context, filter string,
	sort string, selection string) (items []MyData, err error) {

	return c.MySqlPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MyMySqlPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.MySqlPersistence.GetCountByFilter(ctx, filter)
}

func (c *MyMySqlPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.MySqlPersistence.GetOneRandom(ctx, filter)
}

func (c *MyMySqlPersistence) DeleteByFilter(ctx context.Context, filter string) (err error) {
	return c.MySqlPersistence.DeleteByFilter(ctx, filter)
}
```

```
import cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

persistence := NewMyMySqlPersistence()
persistence.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 3306,
	"credential.username", "root",
	"credential.password", "",
	"connection.database", "pip",
))
```

```
err := persistence.Open(context.Background())
```

```
res, err := persistence.Create(context.Background(), data1)
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
result, err := persistence.GetOneRandom(context.Background(), "`key`='key 1'")
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```

```
itemsList, err := persistence.GetListByFilter(context.Background(), "`key`='key 1'", "", "")
```

```
itemsList[0].Id      // Returns '1'
itemsList[0].Key     // Returns 'key 1'
itemsList[0].Content // Returns 'content 1'
```

```
page, err := persistence.GetPageByFilter(context.Background(), "`key`='key 1'", *cquery.NewEmptyPagingParams(), "", "")
```

```
page.Data[0].Id      // Returns '1'
page.Data[0].Key     // Returns 'key 1'
page.Data[0].Content // Returns 'content 1'
```

```
count, err := persistence.GetCountByFilter(context.Background(), "`key`='key 1'")
```

```
err = persistence.DeleteByFilter(context.Background(), "`key`='key 1'")
```

```
import (
      mysqlpersist "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go/persistence"
)
```

```
type MyMySqlPersistence struct {
	*mysqlpersist.IdentifiableJsonMySqlPersistence[MyData, string]
}

func NewMyMySqlPersistence() *MyMySqlPersistence {
	c := &MyMySqlPersistence{}
	c.IdentifiableJsonMySqlPersistence = mysqlpersist.InheritIdentifiableJsonMySqlPersistence[MyData, string](c, "mydata_json")
	return c
}

func (c *MyMySqlPersistence) DefineSchema() {
	c.ClearSchema()
	c.EnsureTable("", "")
	c.EnsureSchema("ALTER TABLE `" + c.TableName + "` ADD `data_key` VARCHAR(50) AS (JSON_UNQUOTE(`data`->\"$.key\"))")
	c.EnsureIndex(c.TableName+"_json_key", map[string]string{"data_key": "1"}, map[string]string{"unique": "true"})
}

func (c *MyMySqlPersistence) GetPageByFilter(ctx context.Context, filter string,
	paging cquery.PagingParams, sort string, selection string) (page cquery.DataPage[MyData], err error) {

	return c.MySqlPersistence.GetPageByFilter(ctx, filter, paging, sort, selection)
}

func (c *MyMySqlPersistence) GetListByFilter(ctx context.Context, filter string,
	sort string, selection string) (items []MyData, err error) {

	return c.MySqlPersistence.GetListByFilter(ctx, filter, sort, selection)
}

func (c *MyMySqlPersistence) GetCountByFilter(ctx context.Context, filter string) (int64, error) {
	return c.MySqlPersistence.GetCountByFilter(ctx, filter)
}

func (c *MyMySqlPersistence) GetOneRandom(ctx context.Context, filter string) (item MyData, err error) {
	return c.MySqlPersistence.GetOneRandom(ctx, filter)
}

func (c *MyMySqlPersistence) DeleteByFilter(ctx context.Context, filter string) (err error) {
	return c.MySqlPersistence.DeleteByFilter(ctx, filter)
}
```

```
result, err := persistence.GetOneRandom(context.Background(), "`key`='key 1'")
```

```
result.Id      // Returns '1'
result.Key     // Returns 'key 1'
result.Content // Returns 'content 1'
```
