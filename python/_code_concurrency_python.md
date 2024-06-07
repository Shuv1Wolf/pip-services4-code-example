```
from pip_services4_logic.state import IStateStore


class MyComponent:
    _store: IStateStore = None

    ...

    def do_something(self, correlation_id: str, object_id: str):
        # Get state from the store or create a new one if the state wasn't found
        state: MyState = self._store.load(correlation_id, "mycomponent:" + object_id)
        if state is not None: state = MyState()
        ...

        self._store.save(correlation_id, "mycomponent:" + object_id, state)

```

```
from pip_services4_logic.cache import ICache


class MyComponent:
    _cache: ICache = None

    ...

    def get_my_object_by_id(self, correlation_id: str, object_id: str) -> MyObject:
        result = self._cache.retrieve(correlation_id, "mycomponent:" + object_id)
        if result is not None:
            return result

        # Retrieve the object
        ...

        self._cache.store(correlation_id, "mycomponent:" + object_id, result, 1000)
        return result
```

```
from pip_services4_logic.lock import ILock


class MyComponent:
    _lock: ILock = None

    ...

    def process_my_object(self, correlation_id: str, object_id: str):
        # Acquire lock for 10 secs
        self._lock.acquire_lock(correlation_id, "mycomponent:" + object_id, 10000, 10000)
        try:
            ...
        finally:
            # Release lock
            self._lock.release_lock(correlation_id, "mycomponent:" + object_id)

```

```
from pip_services4_logic.lock import ILock


class MyComponent:
    _lock: ILock = None

    ...

    def process_my_object(self, correlation_id: str, object_id: str):
        # Try to acquire lock for 10 secs
        if not self._lock.try_acquire_lock(correlation_id, "mycomponent:" + object_id, 10000):

            # Other instance already executing that transaction
            return

        ...
```
