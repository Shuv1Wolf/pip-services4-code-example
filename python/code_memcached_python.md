```python
pip install pip-services4-memcached
```

```python
from pip_services4_memcached.cache import MemcachedCache
```

```python
cache = MemcachedCache()

from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

cache.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 11211
))

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)
```

```python
cache.open(my_context)
```

```python
cache.store(my_context, "key1", "ABC", 5000) # Returns True if successful and False otherwise.

```

```python
value = cache.retrieve(my_context, "key1", )  # Returns "ABC"

```

```python
cache.remove(my_context,'key1') # Returns True if successful and False otherwise

```

```python
from pip_services4_memcached.lock import MemcachedLock
```

```python
lock = MemcachedLock()

from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

lock.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 11211
))

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)
```

```python
lock.open(my_context)
```

```python
lock.try_acquire_lock(my_context, "key3566", 34000) # Returns True is successful and False otherwise
```

```python
lock.acquire_lock(my_context, "key1", 3000, 1000)
```

```python
lock.release_lock(my_context, "key1")
```

```python
from pip_services4_memcached.lock import MemcachedLock
from pip_services4_components.config import ConfigParams
from pip_services4_components.context import Context

lock = MemcachedLock()

lock.configure(ConfigParams.from_tuples(
    "connection.host", "localhost",
    "connection.port", 11211
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

lock.close(my_context)
```
