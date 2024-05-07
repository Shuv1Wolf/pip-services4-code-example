```
import (
	cache "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/cache"
)
```

```
myCache := cache.NewMemoryCache[any]()
```

```
myCache := cache.NewNullCache[any]()
```

```
myCachedValue, _ := myCache.Store(context.Background(), "key1", "ABC", 180000) // Returns "ABC"
```

```
myRetrievedValue, _ := myCache.Retrieve(context.Background(), "key1") // Returns "ABC"
```

```
myCache.Remove(context.Background(), "key1")
```
