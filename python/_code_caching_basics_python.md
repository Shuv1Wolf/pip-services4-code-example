```python
from pip_services4_logic.cache import MemoryCache
```

```python
from pip_services4_logic.cache import NullCache
```

```python
my_cache = MemoryCache()
```

```python
my_cache = NullCache()

```

```python
my_cached_value = my_cache.store(None, "key1", "ABC", 180000)  # Returns "ABC"
```

```python
my_retrieved_value = my_cache.retrieve(None,"key1")   # Returns "ABC"

```

```python
my_cache.remove(None, "key1")

```

```python
my_retrieved_value = my_cache.retrieve(None,"key1")
my_retrieved_value


```
