/build/BeaconsServiceFactory.go
```
package build

import (
	controllers1 "github.com/pip-services-samples/service-beacons-go/controllers/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	logic "github.com/pip-services-samples/service-beacons-go/service"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type BeaconsServiceFactory struct {
	cbuild.Factory
}

func NewBeaconsServiceFactory() *BeaconsServiceFactory {
	c := &BeaconsServiceFactory{
		Factory: *cbuild.NewFactory(),
	}

	memoryPersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "memory", "*", "1.0")
	postgresPersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "postgres", "*", "1.0")
	mongoPersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "mongodb", "*", "1.0")
	filePersistenceDescriptor := cref.NewDescriptor("beacons", "persistence", "file", "*", "1.0")
	serviceDescriptor := cref.NewDescriptor("beacons", "service", "default", "*", "1.0")
	httpcontrollerV1Descriptor := cref.NewDescriptor("beacons", "controller", "http", "*", "1.0")

	c.RegisterType(postgresPersistenceDescriptor, persist.NewBeaconsPostgresPersistence)
	c.RegisterType(mongoPersistenceDescriptor, persist.NewBeaconsMongoPersistence)
	c.RegisterType(memoryPersistenceDescriptor, persist.NewBeaconsMemoryPersistence)
	c.RegisterType(filePersistenceDescriptor, persist.NewBeaconsFilePersistence)
	c.RegisterType(serviceDescriptor, logic.NewBeaconsService)
	c.RegisterType(httpcontrollerV1Descriptor, controllers1.NewBeaconsHttpControllerV1)

	return c
}

```

/containers/BeaconsProcess.go
```
package containers

import (
	factory "github.com/pip-services-samples/service-beacons-go/build"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	rbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	cpg "github.com/pip-services4/pip-services4-go/pip-services4-postgres-go/build"
)

type BeaconsProcess struct {
	cproc.ProcessContainer
}

func NewBeaconsProcess() *BeaconsProcess {
	c := &BeaconsProcess{
		ProcessContainer: *cproc.NewProcessContainer("beacons", "Beacons microservice"),
	}

	c.AddFactory(factory.NewBeaconsServiceFactory())
	c.AddFactory(rbuild.NewDefaultHttpFactory())
	c.AddFactory(cpg.NewDefaultPostgresFactory())

	return c
}

```

/config/config.yml
```
---
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "beacons"
  description: "Beacons microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Tracer that posts records to log
- descriptor: "pip-services:tracer:log:default:1.0"

# Performance counters that post values to log
- descriptor: "pip-services:counters:log:default:1.0"

{{#unless FILE_ENABLED}}{{#unless POSTGRES_ENABLED}}{{#unless MONGO_ENABLED}}
# Memory persistence
- descriptor: "beacons:persistence:memory:default:1.0"
{{/unless}}{{/unless}}{{/unless}}

{{#if FILE_ENABLED}}
# File persistence
- descriptor: "beacons:persistence:file:default:1.0"
  path: {{FILE_PATH}}{{#unless FILE_PATH}}"./data/beacons.json"{{/unless}}
{{/if}}

{{#if POSTGRES_ENABLED}}
# PostgreSQL persistence
- descriptor: "beacons:persistence:postgres:default:1.0"
  connection:
    uri: {{POSTGRES_SERVICE_URI}}
    host: {{POSTGRES_SERVICE_HOST}}{{#unless POSTGRES_SERVICE_HOST}}"localhost"{{/unless}}
    port: {{POSTGRES_SERVICE_PORT}}{{#unless POSTGRES_SERVICE_PORT}}5432{{/unless}}
    database: {{POSTGRES_DB}}{{#unless POSTGRES_DB}}"test"{{/unless}}
  credential:
    username: {{POSTGRES_USER}}{{#unless POSTGRES_USER}}"postgres"{{/unless}}
    password: {{POSTGRES_PASSWORD}}{{#unless POSTGRES_PASSWORD}}"postgres#"{{/unless}}
{{/if}}

{{#if MONGO_ENABLED}}
# MongoDb persistence
- descriptor: "beacons:persistence:mongodb:default:1.0"
  connection:
    uri: {{MONGO_SERVICE_URI}}
    host: {{MONGO_SERVICE_HOST}}{{#unless MONGO_SERVICE_HOST}}"localhost"{{/unless}}
    port: {{MONGO_SERVICE_PORT}}{{#unless MONGO_SERVICE_PORT}}27017{{/unless}}
    database: {{MONGO_DB}}{{#unless MONGO_DB}}"test"{{/unless}}
{{/if}}

# Service
- descriptor: "beacons:service:default:default:1.0"

# Shared HTTP Endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{HTTP_PORT}}{{#unless HTTP_PORT}}8080{{/unless}}

# HTTP controller V1
- descriptor: "beacons:controller:http:default:1.0"
  swagger:
    enable: true

# Hearbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"

# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"

```