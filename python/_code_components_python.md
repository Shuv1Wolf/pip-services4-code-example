```
from typing import Optional

from pip_services4_components.config import IConfigurable, ConfigParams
from pip_services4_components.refer import IReferenceable, IUnreferenceable, IReferences
from pip_services4_components.run import IOpenable, ICleanable
from pip_services4_components.exec import IExecutable, INotifiable, Parameters

class MyComponent(IConfigurable, IReferenceable, IUnreferenceable, IOpenable, IExecutable, INotifiable, ICleanable):

    def __init__(self):
        """Initialize the component"""
    def configure(self, config: ConfigParams):
        """configure the component"""

    def set_references(self, references: IReferences):
        """set component dependencies"""

    def unset_references(self):
        """unset component references"""

    def is_open(self) -> bool:
        """return the component open state"""

    def open(self, correlation_id: Optional[str]):
        """open the component"""

    def close(self, correlation_id: Optional[str]):
        """close the component"""

    def execute(self, correlation_id: Optional[str], args: Parameters):
        """execute the component transaction"""

    def notify(self, correlation_id: Optional[str], args: Parameters):
        """notify the component about events"""

    def clear(self, correlation_id: Optional[str]):
        """clear the component state"""

```

```
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "pip-service-data"
  description: "Entities data microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Performance log counters
- descriptor: "pip-services:counters:log:default:1.0"

{{#if MONGO_ENABLED}}
# MongoDb persistence
- descriptor: "pip-service-data:persistence:mongodb:default:1.0"
  connection:
    uri: {{MONGO_SERVICE_URI}}
    host: {{MONGO_SERVICE_HOST}}{{#unless MONGO_SERVICE_HOST}}"localhost"{{/unless}}
    port: {{MONGO_SERVICE_PORT}}{{#unless MONGO_SERVICE_PORT}}27017{{/unless}}
    database: {{MONGO_DB}}{{#unless MONGO_DB}}"test"{{/unless}}
  credential:
    username: {{MONGO_USER}}
    password: {{MONGO_PASS}}
{{/if}}

{{#unless MONGO_ENABLED}}
# Default to in-memory persistence, if nothing is set
- descriptor: "pip-service-data:persistence:memory:default:1.0"
{{/unless}}

# Service
- descriptor: "pip-service-data:service:default:default:1.0"

{{#if HTTP_ENABLED}}
# Common HTTP endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{HTTP_PORT}}{{#unless HTTP_PORT}}8080{{/unless}}

# HTTP controller version 1.0
- descriptor: "pip-service-data:controller:http:default:1.0"
  swagger:
    enable: true

# Swagger service
- descriptor: "pip-services:swagger-controller:http:default:1.0"
{{/if}}

{{#if GRPC_ENABLED}}
# Common GRPC endpoint
- descriptor: "pip-services:endpoint:grpc:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{GRPC_PORT}}{{#unless GRPC_PORT}}8090{{/unless}}

# GRPC controller version 1.0
- descriptor: "pip-service-data:controller:grpc:default:1.0"
{{/if}}
```

```
from pip_services4_container import ProcessContainer
from pip_services4_grpc.build.DefaultGrpcFactory import DefaultGrpcFactory
from pip_services4_http.build import DefaultRpcFactory
from pip_services4_swagger.build.DefaultSwaggerFactory import DefaultSwaggerFactory


class MyProcess(ProcessContainer):
    def __init__(self):
        super().__init__("mymicroservice", "Sample microservice container")

        self._factories.add(MyComponentFactory())
        self._factories.add(DefaultRpcFactory())
        self._factories.add(DefaultSwaggerFactory())
        self._factories.add(DefaultGrpcFactory())
```
