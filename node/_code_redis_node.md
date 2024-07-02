```
 npm install pip-services4-redis-node --save
```

```
import { RedisCache } from "pip-services4-redis-node";
```

```
import { RedisCache } from "pip-services4-redis-node";
import { ConfigParams } from 'pip-services4-components-node';

let cache = new RedisCache();
cache.configure(ConfigParams.fromTuples(
    "connection.host", "localhost",
    "connection.port", 6379
));
```

```
await cache.open(ctx);

```

```
await cache.close(ctx);

```

```
let result = await cache.store(ctx, "key1", "ABC", 10000); // Returns "OK" if successful 

```

```
result = await cache.retrieve(ctx, "key1");  // Returns "ABC"

```

```
await cache.remove(ctx, "key1");

```

```
import { RedisLock  } from "pip-services4-redis-node";
```

```
import { RedisLock  } from "pip-services4-redis-node";
import { ConfigParams } from 'pip-services4-components-node';

let lock = new RedisLock();
lock.configure(ConfigParams.fromTuples(
    "connection.host", "localhost",
    "connection.port", 6379
));
```

```
await lock.open(ctx);

```

```
await lock.close(ctx);

```

```
let locked = await lock.tryAcquireLock(ctx, "key1", 3300);

```

```
await lock.acquireLock(ctx, "key1", 3000, 1000);

```

```
await lock.releaseLock(ctx, "key1");

```

```
import { RedisLock  } from "pip-services4-redis-node";
import { ConfigParams } from 'pip-services4-components-node';

let lock = new RedisLock();
lock.configure(ConfigParams.fromTuples(
    "connection.host", "localhost",
    "connection.port", 6379
));

await lock.open(ctx);
await lock.acquireLock(ctx, "key1", 3000, 1000);

try {
    // Processing...
} finally {
    await lock.releaseLock(ctx, "key1");
}
```
