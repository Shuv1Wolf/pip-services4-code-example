```python
from pip_services4_data.data import IStringIdentifiable

class Dummy(IStringIdentifiable): 
    def __init__(self, id: str = None, key: str = None, content: str = None): 
        self.id = id 
        self.key = key 
        self.content = content 
      
dummy1 = Dummy('1', 'key 1', 'content 1') 
dummy2 = Dummy('id 1', 'key 2', 'content 2') 
dummy3 = Dummy(None, 'key 3', 'content 3')
```

```python
from typing import Callable, Optional, Any 
from pip_services4_persistence.persistence import IdentifiableMemoryPersistence 
from pip_services4_data.query import FilterParams, PagingParams, DataPage 
from pip_services4_components.context import Context

class MyMemoryPersistence(IdentifiableMemoryPersistence): 
    def __init__(self): 
        super(MyMemoryPersistence, self).__init__() 
 
    def __compose_filter(self, filter_params: FilterParams) -> Callable[[Dummy], bool]: 
        filter_params = filter_params or FilterParams() 
        id = filter_params.get_as_nullable_string("id") 
        temp_ids = filter_params.get_as_nullable_string("ids") 
        ids = temp_ids.split(",") if temp_ids is not None else None 
        key = filter_params.get_as_nullable_string("key") 
 
        def inner(item: Dummy) -> bool: 
            if id is not None and item.id != id: 
                return False 
            if ids is not None and item.id in ids: 
                return False 
            if key is not None and item.key != key: 
                return False 
            return True 
 
        return inner 
 
    def get_page_by_filter(self, ctx: Context, filter: Any, paging: PagingParams, sort: Any = None, 
                           select: Any = None) -> DataPage: 
        return super().get_page_by_filter(ctx, self.__compose_filter(filter), paging, sort, select) 
 
    def get_one_by_key(self, ctx: Context, key): 
        for item in self._items: 
            if item.key == key: 
                self._logger.trace(ctx, "Found object by key={}", key) 
                return item 
           
        self._logger.trace(ctx, "Cannot find by key={}", key) 
 

persistence = MyMemoryPersistence()
```

```python
result = persistence.create(None, dummy1) 
print(f'Created Dummy with ID: {result.id}')
 
result = persistence.create(None, dummy2) 
print(f'Created Dummy with ID: {result.id}')

result = persistence.create(None, dummy3) 
print(f'Created Dummy with ID: {result.id}') 
```

```python
# get one item 
result = persistence.get_page_by_filter(None, 
                                        FilterParams.from_tuples('key', 'key 2'), 
                                        PagingParams(0, None,True)) 
print(f'Has {result.total} items') 
for item in result.data: 
    print(f'{item.id}, {item.key}, {item.content}') 
```

```python
# get all items 
result = persistence.get_page_by_filter(None, 
                                        None, 
                                        PagingParams(0, None, True)) 
print(f'Has {result.total} items') 
for item in result.data: 
    print(f'{item.id}, {item.key}, {item.content}')
```

```python
result = persistence.update(None, Dummy('id 1', 'key 2', 'new content 2') )
```

```python

# get all items 
result = persistence.get_page_by_filter(None, 
                                        FilterParams.from_tuples('id', 'id 1'), 
                                        PagingParams(0, 3)) 

for item in result.data: 
    print(f'{item.id}, {item.key}, {item.content}')
```

```python
result = persistence.update_partially(None, 'id 1', {'content' : 'new new content 2'})

```

```python
# get all items 
result = persistence.get_page_by_filter(None, 
                                        FilterParams.from_tuples('id', 'id 1'), 
                                        PagingParams(0, None, True)) 

for item in result.data: 
    print(f'{item.id}, {item.key}, {item.content}')
```

```python
result = persistence.delete_by_id(None, "1") 
```

```python

# get all items 
result = persistence.get_page_by_filter(None, 
                                        FilterParams.from_tuples('id', '1'), 
                                        PagingParams(0, None, True)) 
print(f'Has {result.total} items') 
```
