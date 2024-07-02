```
import { Context } from "pip-services4-components-node";
import { IStateStore } from "pip-services4-logic-node";

class MyComponent {
    private _store: IStateStore;

    ...

    public doSomething(ctx: Context, objectId: string) {
        // Get state from the store or create a new one if the state wasn't found
        let state: MyState = await this._store.load(ctx, "mycomponent:" + objectId);
        if (state != null) { state = new MyState(); }
        ...

        await this._store.save(ctx, "mycomponent:" + objectId, state);
    }
}

```

```
import { Context } from "pip-services4-components-node";
import { ICache } from "pip-services4-logic-node";

class MyComponent {
    private _cache: ICache;
  
    ...
  
    public getMyObjectById(ctx: Context, objectId: string): Promise<MyObject> {
      let result = await this._cache.retrieve(ctx, "mycomponent:" + objectId);
      if (result != null) { return result; }
  
      // Retrieve the object
      ...
  
      await this._cache.store(ctx, "mycomponent:" + objectId, result, 1000);
      return result;
    }
  }
```

```
import { Context } from "pip-services4-components-node";
import { ILock } from "pip-services4-logic-node";

class MyComponent {
    private _lock: ILock;
  
    ...
    public processMyObject(ctx: Context, objectId: string) {
      // Acquire lock for 10 secs
      await this._lock.acquireLock(ctx, "mycomponent:" + objectId, 10000, 10000);
      try {
        ...
      } finally {
        // Release lock
        await this._lock.releaseLock(ctx, "mycomponent:" + objectId);
      }
    }
  }
```

```
import { Context } from "pip-services4-components-node";
import { ILock } from "pip-services4-logic-node";

class MyComponent {
    private _lock: ILock;
  
    ...
    public ProcessMyObject(ctx: Context, objectId: string)
    {
        // Try to acquire lock for 10 secs
        if (!this._lock.tryAcquireLock(ctx, "mycomponent:" + objectId, 10000))
        {
            // Other instance already executing that transaction
            return;
        }

        ...
    }
  }
```
