### Validation

```go
import (
	"context"

	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"

	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type MyObjectSchema struct {
	*cvalid.ObjectSchema
}

func NewMyObjectSchema() *MyObjectSchema {
	c := MyObjectSchema{}
	c.ObjectSchema = cvalid.NewObjectSchema()

	c.WithOptionalProperty("prop1", cconv.Integer)
	c.WithRequiredProperty("prop2", cconv.String)
	c.WithOptionalProperty("prop3", NewMySubObjectSchema())
	return &c
}
```

### Filtering

```go
filter := cquery.NewFilterParamsFromTuples(
		"key1", "ABC",
		"key2", 123,
	)

values := client.GetMyObjects(context.Background(), filter)
```

### Sorting

```go
sorting := cquery.NewSortParams([]cquery.SortField{
		cquery.NewSortField("key1", true),
		cquery.NewSortField("key2", false),
	})

values := client.GetMyObjects(context.Background(), filter, sorting)
```

### Paging

```
paging := cquery.NewPagingParams(0, 100, true)
page := client.GetMyObjects(context.Background(), filter, sorting, paging)
```

### Projections

```
projection := cquery.NewProjectionParamsFromStrings([]string{"field1", "field2"})
page := client.GetMyObjects(context.Background(), filter, sorting, paging, projection)
```
