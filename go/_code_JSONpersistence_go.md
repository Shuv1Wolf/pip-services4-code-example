```
import (
	persist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)
```

```
import (
	persist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)

persister := persist.NewJsonFilePersister[any]("./data.json")
```

```
import (
	"context"

	config "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	persist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)

persister := persist.NewJsonFilePersister[any]("")
myConfig := config.NewConfigParamsFromTuples("path", "./data.json")
persister.Configure(context.Background(), myConfig)
```

```
persister.Save(context.Background(), []interface{}{"A1", "B1", "C1"})
```

```
items, _ := persister.Load(context.Background())
```

```
import (
	"context"
	"fmt"

	persist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)

// Create the JSON persistence component
persister := persist.NewJsonFilePersister[any]("./data.json")
// Save data on the JSON persistence object
persister.Save(context.Background(), []interface{}{"A1", "B1", "C1"})
// Read data from the JSON persistence object
items, _ := persister.Load(context.Background())

fmt.Print(items)
```

```

```
