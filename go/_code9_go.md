### State management

```
import (
	"context"

	cstate "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/state"
)

type MyComponent struct {
	store cstate.IStateStore[any]
}

// ...

func (c *MyComponent) DoSomething(ctx context.Context, objectId string) {
	// Get state from the store or create a new one if the state wasn't found
	state := c.store.Load(ctx, "mycomponent:"+objectId)
	if state != nil {
		// ...
	}

	c.store.Save(ctx, "mycomponent:"+objectId, state)
}
```

### Caching

```
import (
	"context"

	ccache "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/cache"
)

type MyComponent struct {
	cache ccache.ICache[any]
}

// ...

func (c *MyComponent) GetMyObjectById(ctx context.Context, objectId string) any {
	// Get state from the store or create a new one if the state wasn't found
	result, err := c.cache.Retrieve(ctx, "mycomponent:"+objectId)
	if err != nil {
		return err
	}
	if result != nil {
		return result
	}

	// Retrieve the object
	// ...

	c.cache.Store(ctx, "mycomponent:"+objectId, result, 1000)
	return result
}
```

### Locking

```
import (
	"context"

	clock "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/lock"
)

type MyComponent struct {
	lock clock.ILock
}

// ...

func (c *MyComponent) ProcessMyObject(ctx context.Context, objectId string) {
	// Acquire lock for 10 secs
	c.lock.AcquireLock(ctx, "mycomponent:"+objectId, 10000, 10000)
	// Release lock
	defer c.lock.ReleaseLock(ctx, "mycomponent:"+objectId)

	// ...
}
```

```
import (
	"context"

	clock "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/lock"
)

type MyComponent struct {
	lock clock.ILock
}

// ...

func (c *MyComponent) ProcessMyObject(ctx context.Context, objectId string) error {
	// Try to acquire lock for 10 secs
	ok, err := c.lock.TryAcquireLock(ctx, "mycomponent:"+objectId, 10000)
	if !ok || err != nil {
		// Other instance already executing that transaction
		return err
	}

	// ...
}
```
