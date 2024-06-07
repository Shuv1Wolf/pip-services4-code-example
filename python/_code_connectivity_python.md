```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
connection:
  host: mongo
  port: 27017

```

```
from pip_services4_components.config import ConfigParams, IConfigurable
from pip_services4_config.connect import ConnectionParams

class MyPersistence(IConfigurable):
    _host: str
    _port: int

    def configure(self, config: ConfigParams):
        connection = ConnectionParams.from_config(config.get_section("connection"))
        self._host = connection.get_host()
        self._port = connection.get_port_with_default(27017)
```

```
# In-memory discovery service
descriptor: "pip-services:discovery:memory:default:1.0"
mongo:
  host: mongo
  port: 27017

# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
connection:
  discovery_key: mongo
```

```
from pip_services4_components.config import ConfigParams, IConfigurable
from pip_services4_components.refer import IReferenceable, IReferences
from pip_services4_config.auth import CredentialResolver


class MyPersistence(IConfigurable, IReferenceable):
    ...
    _credential_resolver = CredentialResolver()
    _username: str
    _password: str

    def configure(self, config: ConfigParams):
        ...
        self._connection_resolver.configure(config)

    def set_references(self, refs: IReferences):
        ...
        self._credential_resolver.set_references(refs)

        credentials = self._credential_resolver.lookup(None)
        self._username = credentials.get_username()
        self._password = credentials.get_password()
```

```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
credential:
  username: admin
  password: pass123

```

```
from pip_services4_components.config import ConfigParams, IConfigurable
from pip_services4_config.auth import CredentialParams


class MyPersistence(IConfigurable):
    ...
    _username: str
    _password: str

    def configure(self, config: ConfigParams):
        ...
        credentials = CredentialParams.from_config(config.get_section("credential"))
        self._username = credentials.get_username()
        self._password = credentials.get_password()
```

```
# In-memory credential store
descriptor: "pip-services:credential-store:memory:default:1.0"
mongo:
  username: admin
  password: pass123

# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
connection:
  ...
credential:
  discovery_key: mongo

```

```
from pip_services4_components.config import ConfigParams, IConfigurable
from pip_services4_components.refer import IReferenceable, IReferences
from pip_services4_config.connect import ConnectionResolver


class MyPersistence(IConfigurable, IReferenceable):
    _connectionResolver = ConnectionResolver()
    _host: str
    _port: int

    def configure(self, config: ConfigParams):
        self._connectionResolver.configure(config)

    def set_references(self, refs: IReferences):
        self._connectionResolver.set_references(refs)

        connection = self._connectionResolver.resolve(None)
        self._host = connection.get_host()
        self._port = connection.get_port_with_default(27017)
```
