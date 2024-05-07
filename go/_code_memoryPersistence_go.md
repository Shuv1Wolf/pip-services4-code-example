```
package persistence

type Dummy struct {
	Id      string `json:"id"`
	Key     string `json:"key"`
	Content string `json:"content"`
}


dummy1 := Dummy{
	Id: "1",
	Key: "key 1",
	Content: "content 1",
}
dummy2 := Dummy{
	Id: "id 1",
	Key: "key 2",
	Content:  "content 2",
}
dummy3 := Dummy{
	Id: "",
	Key: "key 3",
	Content: "content 3",
}
```

```
import (
	"context"
	"fmt"
	"strings"

	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cpersist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)

type Dummy struct {
	Id      string `json:"id"`
	Key     string `json:"key"`
	Content string `json:"content"`
}

type MyMemoryPersistence struct {
	*cpersist.IdentifiableMemoryPersistence[Dummy, string]
}

func NewMyMemoryPersistence() *MyMemoryPersistence {
	return &MyMemoryPersistence{IdentifiableMemoryPersistence: cpersist.NewIdentifiableMemoryPersistence[Dummy, string]()}
}

func composeFilter(filter *cquery.FilterParams) func(item Dummy) bool {
	if filter == nil {
		filter = cquery.NewFilterParams(make(map[string]string))
	}

	id, _ := filter.GetAsNullableString("id")
	temp_ids, idsOk := filter.GetAsNullableString("ids")

	var ids []string
	if idsOk {
		ids = strings.Split(temp_ids, ",")

	}

	key, _ := filter.GetAsNullableString("key")

	return func(dummy Dummy) bool {
		if id != "" && dummy.Id != id {
			return false
		}
		if key != "" && dummy.Key != key {
			return false
		}

		if len(ids) > 0 {
			for _, v := range ids {
				if dummy.Id == v {
					return true
				}
			}
			return false
		}
		return true
	}
}

func (c *MyMemoryPersistence) GetOneByKey(ctx context.Context, key string) (item Dummy, err error) {
	for _, val := range c.Items {
		if val.Key == key {
			item = val
			break
		}
	}
	return item, err
}

func (c *MyMemoryPersistence) GetPageByFilter(ctx context.Context, filter *cquery.FilterParams, paging *cquery.PagingParams) (page cquery.DataPage[Dummy], err error) {

	if &filter == nil {
		filter = cquery.NewEmptyFilterParams()
	}

	tempPage, err := c.IdentifiableMemoryPersistence.GetPageByFilter(ctx, composeFilter(filter), *paging, nil, nil)

	return tempPage, err
}

func NewDummyPage(len *int64, data []Dummy) cquery.DataPage[Dummy] {
	return cquery.DataPage[Dummy]{Total: int(*len), Data: data}
}

// ...

persistence := NewMyMemoryPersistence()
```

```
item, _ := persistence.Create(context.Background(), dummy1)
fmt.Println("Created Dummy with ID: " + item.Id)

item, _ = persistence.Create(context.Background(), dummy2)
fmt.Println("Created Dummy with ID: " + item.Id)

item, _ = persistence.Create(context.Background(), dummy3)
fmt.Println("Created Dummy with ID: " + item.Id)
```

```
page, _ := persistence.GetPageByFilter(context.Background(), cquery.NewFilterParamsFromTuples("id", "id 1", "key", "key 2"), cquery.NewPagingParams(0, 1, true))

fmt.Printf("Has %v items \n", page.Total)
for _, v := range page.Data {
	fmt.Printf("%v, %v, %v \n", v.Id, v.Key, v.Content)
}
```

```
// all items

items := persistence.Items
for _, v := range items {
	fmt.Printf("%v, %v, %v \n", v.Id, v.Key, v.Content)
}
```

```
result, _ := persistence.Update(context.Background(), Dummy{"id 1", "key 2", "new content 2"})
```

```
import (
    cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
)

// update patially
updateMap := cdata.NewAnyValueMap(map[string]interface{}{"content": "new new content 2"})
item, _ = persistence.UpdatePartially(context.Background(), "id 1", *updateMap)

fmt.Printf("%v , %v, %v \n", item.Id, item.Key, item.Content)
```

```
result, _ = persistence.DeleteById(context.Background(), "1")
```
