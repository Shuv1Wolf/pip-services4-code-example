```
pip install pip-services4-redis
```

```
from pip_services4_redis.cache import RedisCache
```

```
from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

cache = RedisCache()
cache.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 6379
))

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)
```

```
cache.open(my_context)
```

```
cache.close(my_context)
```

```
result = cache.store(my_context, "key1", "ABC", None)  # Returns True if successful and False otherwise 
```

```
result = cache.retrieve(my_context, 'key1')  # Returns b'ABC'
```

```
result = cache.remove(my_context,'key1') # Returns 1 if successful and 0 otherwise
```

```
from pip_services4_redis.lock import RedisLock
```

```
from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

lock = RedisLock()
lock.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 6379
))

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)
```

```
lock.open(my_context)
```

```
lock.close(my_context)
```

```
locked = lock.try_acquire_lock(my_context, "key1", 3300) # returns bool
```

```
lock.acquire_lock(my_context, "key1", 3000, 1000)
```

```
lock.release_lock(my_context, "key1")
```

```
from pip_services4_redis.lock import RedisLock
from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

lock = RedisLock()
lock.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 6379
))

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)

lock.open(my_context)
lock.acquire_lock(my_context, "key1", 3000, 1000)

try:
    # Processing...
    pass
finally:
    lock.release_lock(my_context, "key1")
```
