```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	postgres "github.com/pip-services4/pip-services4-go/pip-services4-postgres-go"
	mysql "github.com/pip-services4/pip-services4-go/pip-services4-mysql-go"
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

```
