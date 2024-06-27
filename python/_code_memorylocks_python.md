```
from pip_services4_logic.lock.MemoryLock import MemoryLock
```

```
lock = MemoryLock()
```

```
from pip_services4_components.config import ConfigParams

lock = MemoryLock()

config = ConfigParams.from_tuples("retry_timeout", 200)
lock.configure(config)
```

```
from pip_services4_components.context import Context

context_data = {
  "traceId": "123",
}

my_context = Context(context_data)

lock.acquire_lock(my_context, "mykey", 1000, 1000, )
```

```

lock.release_lock(my_context, "mykey")
```

```
from pip_services4_components.refer import Descriptor, References, IReferences, IReferenceable
from pip_services4_logic.cache import ICache, MemoryCache
from pip_services4_logic.lock.ILock import ILock
from pip_services4_logic.lock.MemoryLock import MemoryLock
from pip_services4_components.config import ConfigParams

class MyComponent(IReferenceable):
    __cache: ICache
    __lock: ILock

    def set_references(self, references: IReferences):
        self.__cache = references.get_one_required(Descriptor("*", "cache", "*", "*", "1.0"))
        self.__lock = references.get_one_required(Descriptor("*", "lock", "*", "*", "1.0"))

    def store_result(self, context, param1):    

        print("The stored value is " + param1)

        # Lock
        self.__lock.acquire_lock(context, "mykey", 1000, 1000, )
      
        config = ConfigParams.from_tuples("retry_timeout", 200)
        self.__lock.configure(config)
      
        # Do processing
        # ...
     

        # Store result to cache async
        self.__cache.store(context, 'mykey', param1, 3600000)
  
        # Release lock async
        self.__lock.release_lock(context, 'mykey')

      
    def obtain_result(self, context):
      
         # Lock..
        self.__lock.acquire_lock(context, "mykey", 1000, 1000, )
      
        # Do processing
        # ...
        result = self.__cache.retrieve(context, "mykey")
  
        # Release lock async
        self.__lock.release_lock(context, "mykey")
  
        return result
  
  
# Use the component
my_component = MyComponent()
my_component.set_references(References.from_tuples(
    Descriptor("pip-services", "cache", "memory", "default", "1.0"), MemoryCache(),
    Descriptor("pip-services", "lock", "memory", "default", "1.0"), MemoryLock(),
))

my_component.store_result(None, "param1")

result = my_component.obtain_result(None)

print("The retrieved value is " + result)
```

```
from pip_services4_logic.lock.NullLock import NullLock
```

```
lock = NullLock()
```

```
from pip_services4_components.refer import Descriptor, References, IReferences, IReferenceable
from pip_services4_logic.cache import ICache, MemoryCache
from pip_services4_logic.lock.ILock import ILock
from pip_services4_logic.lock.NullLock import NullLock


class MyComponent(IReferenceable):
    __cache: ICache
    __lock: ILock

    def set_references(self, references: IReferences):
        self.__cache = references.get_one_required(Descriptor("*", "cache", "*", "*", "1.0"))
        self.__lock = references.get_one_required(Descriptor("*", "lock", "*", "*", "1.0"))

    def store_result(self, correlation_id, param1):    

        # Lock
        self.__lock.acquire_lock(correlation_id, "mykey", 1000, 1000, )
      
        # Do processing
        # ...
        print("The stored value is " + param1)

        # Store result to cache async
        self.__cache.store(correlation_id, 'mykey', param1, 3600000)
  
        # Release lock async
        self.__lock.release_lock(correlation_id, 'mykey')

      
    def obtain_result(self, correlation_id):
      
         # Lock..
        self.__lock.acquire_lock(correlation_id, "mykey", 1000, 1000, )
      
        # Do processing
        # ...
        result = self.__cache.retrieve(correlation_id, "mykey")
  
        # Release lock async
        self.__lock.release_lock(correlation_id, "mykey")
  
        return result
  
  
# Use the component
my_component = MyComponent()
my_component.set_references(References.from_tuples(
    Descriptor("pip-services", "cache", "memory", "default", "1.0"), MemoryCache(),
    Descriptor("pip-services", "lock", "null", "default", "1.0"), NullLock(),
))

my_component.store_result(None, "param1")

result = my_component.obtain_result(None)

print("The retrieved value is " + result)
```
