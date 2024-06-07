```
# Database connection
descriptor: pip-services:connection:mongodb:default:3.0
connection:
  host: ${{MONGO_HOST}}{$unless MONGO_HOST}localhost{$unless}
  port: ${{MONGO_PORT}}{$unless MONGO_PORT}27017{$unless}
credentials:
  host: ${{MONGO_USER}}{$unless MONGO_USER}mongo{$unless}
  port: ${{MONGO_PASS}}{$unless MONGO_PASS}pwd123{$unless}
options:
  max_pool_size: 10
  keep_alive: true

```

```
# Persistence with default connection
descriptor: myservice:mypersistence:mongodb:persist1:1.0

# Persistence linked to specific connection
descriptor: myservice:mypersistence:mongodb:persist2:1.0
references:
  connection: pip-services:connection:mongodb:conn1:3.0

```

```
from pip_services4_mongodb.persistence import MongoDbPersistence


class MyObject:
    def __init__(self, key: str = None, name: str = None):
        self.name = key
        self.content = name


class MyMongoDbPersistence(MongoDbPersistence):

    def __init__(self):
        super(MyMongoDbPersistence, self).__init__('mycollection')

    def get_by_name(self, correlation_id: str, name: str) -> MyObject:
        criteria = {'name': name}
        res = self.get_list_by_filter(correlation_id, criteria, None, None)
        return None if len(res) < 0 else res[0]

    def create_default(self, correlation_id: str, name: str) -> MyObject:
        name = name or 'unknown'
        key = name.lower().replace(" #$%^&", "_")
        item = MyObject(key, name)

        result = self._collection.insert_one(item)
        item = self._collection.find_one({'_id': result.inserted_id})

        item = self._convert_to_public(item)
        return item

    def delete_by_name(self, correlation_id: str, name: str):
        criteria = {'name': name}
        self.delete_by_filter(correlation_id, criteria)
```

```
from pip_services4_data.data import IIdentifiable

class MyIdentifiableObject(IIdentifiable):
    def __init__(self, id: str = None, name: str = None, value: str = None):
        self.id = id
        self.name = name
        self.value = value
```

```
from abc import ABC

from pip_services4_data.data import IIdentifiable
from pip_services4_postgres.persistence import IdentifiablePostgresPersistence
from pip_services4_data.query import FilterParams, PagingParams, DataPage


class MyIdentifiableObject(IIdentifiable):
    def __init__(self, id: str = None, name: str = None, value: str = None):
        self.id = id
        self.name = name
        self.value = value


class MyIdentifiablePersistence(ABC):
    def get_page_by_filter(self, correlation_id: str, filter: FilterParams, paging: PagingParams) -> DataPage:
        pass

    def create(self, correlation_id: str, item: MyIdentifiableObject) -> MyIdentifiableObject:
        pass

    def get_one_by_id(self, correlation_id: str, id: str) -> MyIdentifiableObject:
        pass

    def delete_by_id(self, correlation_id: str, id: str) -> MyIdentifiableObject:
        pass


class MyIdentifiablePostgreSqlPersistence(IdentifiablePostgresPersistence, MyIdentifiablePersistence):
    def __init__(self):
        super(MyIdentifiablePostgreSqlPersistence, self).__init__('mycollection')

    def _compose_filter(self, filter_params: FilterParams):
        filter_params = filter_params or FilterParams()
        criteria = []

        id = filter_params.get_as_string("id")
        if id is not None and id != "":
            criteria.append("id='" + id + "'")

        name = filter_params.get_as_string("name")
        if name is not None and name != "":
            criteria.append("name='" + name + "'")

        return None if len(criteria) < 0 else " AND ".join(criteria)

    def get_page_by_filter(self, correlation_id: str, filter: FilterParams, paging: PagingParams) -> DataPage:
        criteria = self._compose_filter(filter)
        return super(MyIdentifiablePostgreSqlPersistence, self).get_page_by_filter(correlation_id, criteria, paging, None, None)
```

```
CREATE TABLE MyIdentifiableJsonObject (
  id VARCHAR(32) PRIMARY KEY,
  data JSON
);
```

```
from typing import Optional

from pip_services4_components.config import IConfigurable, ConfigParams
from pip_services4_components.refer import IReferenceable, IReferences
from pip_services4_components.run import IOpenable


class MyCustomPersistence:
    # Custom implementation using any persistence framework
    pass

class MyCustomPersistenceWrapper(IConfigurable, IReferenceable, IOpenable):

    def __init__(self):
        self._config = ConfigParams()
        self._persistence: MyCustomPersistence = None

    def configure(self, config: ConfigParams):
        # Store config parameters
        self._config = config or self._config

    def set_references(self, references: IReferences):
        # Retrieve whatever references you may need
        pass

    def is_open(self) -> bool:
        return self._persistence is not None

    def open(self, correlation_id: Optional[str]):
        if self._persistence is not None:
            return

        # Create custom persistence
        self._persistence = MyCustomPersistence()

        # Configure custom persistence
        # ...

        # Open and connect to the database
        self._persistence.connect()

    def close(self, correlation_id: Optional[str]):
        if self._persistence is None:
            return

        # Disconnect from the database and close
        self._persistence.disconnect()
        self._persistence = None

    def custom_method(self, ...):
        # Delegate operations to custom persistence
        return self._persistence.custom_method(...)
```
