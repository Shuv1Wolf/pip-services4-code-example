```
import { MemoryCache } from 'pip-services4-logic-node';
```

```
import { NullCache } from 'pip-services4-logic-node';
```

```
let myCache = new MemoryCache();
```

```
let myCache = new NullCache();

```

```
let myCachedValue = await myCache.store(ctx, "key1", "ABC", 180000);  // Returns "ABC"

```

```
let myRetrievedValue = await myCache.retrieve(ctx, "key1");  // Returns "ABC"

```

```
await myCache.RemoveAsync(ctx, "key1");

```

```

```
