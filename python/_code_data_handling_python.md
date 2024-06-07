```python
from pip_services4_commons.convert import TypeCode
from pip_services4_data.validate import ObjectSchema


class MyObjectSchema(ObjectSchema):
    def __init__(self):
        super().__init__()

        self.with_required_property('prop1', TypeCode.Integer)
        self.with_optional_property('prop2', TypeCode.String)
        self.with_optional_property('prop3', MySubObjectSchema())
```

```python
from pip_services4_data.query import FilterParams

filter = FilterParams.from_tuples(
    'key1', 'ABC',
    'key2', 123
)

values = client.get_my_objects(filter)
```

```python
from pip_services4_data.query import SortParams
from pip_services4_data.query import SortField

sorting = SortParams(
    SortField("key1", True),
    SortField("key2", False)
)

values = client.get_my_objects(filter, sorting)
```

```python
from pip_services4_data.query import PagingParams

paging = PagingParams(0, 100, True)

page = client.get_my_objects(filter, sorting, paging)
```

```python
from pip_services4_data.query import ProjectionParams

projection = ProjectionParams(['field1', 'field2'])

page = client.getMyObjects(filter, sorting, paging, projection)
```
