```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-redis-go@latest
```

```
import (
    rcache "github.com/pip-services4/pip-services4-go/pip-services4-redis-go/cache"
)
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	rcache "github.com/pip-services4/pip-services4-go/pip-services4-redis-go/cache"
)

cache := rcache.NewRedisCache[string]()
cache.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 6379,
))
```

```
err := cache.Open(context.Background())
```

```
err = cache.Close(context.Background())
```

```
result, err := cache.Store(context.Background(), "key", "ABC", 10000) // Returns saved value and err or nil
```

```
res, err := cache.Retrieve(context.Background(), "key1") // Returns "ABC"
```

```
err := cache.Remove(context.Background(), "key1")
```

```
import (
    rlock "github.com/pip-services4/pip-services4-go/pip-services4-redis-go/lock"
)
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	rlock "github.com/pip-services4/pip-services4-go/pip-services4-redis-go/lock"
)


lock := rlock.NewRedisLock()
lock.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 6379,
))
```

```
err := lock.Open(context.Background())
```

```
err = lock.Close(context.Background())
```

```
locked, err := lock.TryAcquireLock(context.Background(), "key1", 3300)
```

```
err = lock.AcquireLock(context.Background(), "key1", 3000, 1000)
```

```
err = lock.ReleaseLock(context.Background(), "key1")
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	rlock "github.com/pip-services4/pip-services4-go/pip-services4-redis-go/lock"
)

func main() {
	lock := rlock.NewRedisLock()
	lock.Configure(context.Background(), conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 6379,
	))
	err := lock.Open(context.Background())

	err = lock.AcquireLock(context.Background(), "key1", 3000, 1000)

	defer lock.ReleaseLock(context.Background(), "key1")

	// Processing...
}
```
