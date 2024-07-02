```
npm install pip-services4-prometheus-node --save
```

```
import { MemcachedCache } from "pip-services4-memcached-node";
```

```
import { ConfigParams } from "pip-services4-components-node";
import { MemcachedCache } from "pip-services4-memcached-node";

let cache = new MemcachedCache();

cache.configure(ConfigParams.fromTuples(
    "connection.host", "localhost",
    "connection.port", 11211
));
```

```
await cache.open(ctx);
```

```
await cache.store(ctx, "key1", "ABC", 5000); // Returns True if successful and False otherwise.
```

```
var value = await cache.RetrieveAsync<string>(ctx, "key1"); // Returns "ABC"

```

```
await cache.remove(ctx, "key1"); // Returns True if successful and False otherwise

```

```
import { MemcachedLock } from "pip-services4-memcached-node";
```

```
import { ConfigParams, IContext } from "pip-services4-components-node";
import { MemcachedLock } from "pip-services4-memcached-node";

let lock = new MemcachedLock();

lock.configure(ConfigParams.fromTuples(
    "connection.host", "localhost",
    "connection.port", 11211
));
```

```
await lock.open(ctx);

```

```
await lock.tryAcquireLock(ctx, "key3566", 34000); // Returns True is successful and False otherwise

```

```
await lock.acquireLock(ctx, "key1", 3000, 1000);

```

```
await lock.releaseLock(ctx, "key1");

```

```
import { ConfigParams, IContext } from "pip-services4-components-node";
import { MemcachedLock } from "pip-services4-memcached-node";

export async function main() {

  var lock = new MemcachedLock();

  lock.configure(ConfigParams.fromTuples(
      "connection.host", "localhost",
      "connection.port", 11211
  ));

  // ...

  await lock.open(ctx);
  await lock.acquireLock(ctx, "key1", 3000, 1000);

  try {
      // Processing...
  }
  finally {
      await lock.releaseLock(ctx, "key1");
  }

  await lock.close(ctx);
}
```
