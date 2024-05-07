```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-memcached-go@latest
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	memcache "github.com/pip-services4/pip-services4-go/pip-services4-memcached-go/cache"
)


cache := memcache.NewMemcachedCache[any]()

cache.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 11211,
))
```

```
cache.Open(context.Background())
```

```
res, err := cache.Store(context.Background(), "key1", "ABC", 5000)
```

```
value, err := cache.Retrieve(context.Background(), "key1") // Returns "ABC"
```

```
cache.Remove(context.Background(), "key1")   // Retruns: an error or nil for success
```

```
import (
    memlock "github.com/pip-services4/pip-services4-go/pip-services4-memcached-go/lock"
)
```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	memlock "github.com/pip-services4/pip-services4-go/pip-services4-memcached-go/lock"
)


lock := memlock.NewMemcachedLock()

lock.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"connection.host", "localhost",
	"connection.port", 11211,
))
```

```
lock.Open(context.Background())
```

```
lock.TryAcquireLock(context.Background(), "key3566", 34000) // Returns True is successful and False otherwise and err obj
```

```
lock.AcquireLock(context.Background(), "key1", 3000, 1000)
```

```
lock.ReleaseLock(context.Background(), "key1")
```

````
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	memlock "github.com/pip-services4/pip-services4-go/pip-services4-memcached-go/lock"
)

func main() {
	lock := memlock.NewMemcachedLock()
	lock.Configure(context.Background(), conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 11211,
	))

	lock.Open(context.Background())
	defer lock.Close(context.Background())

	lock.AcquireLock(context.Background(), "key1", 3000, 1000)
	defer lock.ReleaseLock(context.Background(), "key1")

	// Processing...
}
````
