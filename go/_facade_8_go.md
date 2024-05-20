```
package build

import (
	service1 "github.com/pip-services-samples/pip-samples-facade-go/services/version1"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type FacadeFactory struct {
	cbuild.Factory
	FacadeServiceV1Descriptor *cref.Descriptor
}

func NewFacadeFactory() *FacadeFactory {

	c := FacadeFactory{
		Factory: *cbuild.NewFactory(),
	}
	c.FacadeServiceV1Descriptor = cref.NewDescriptor("pip-facades-example", "controller", "http", "*", "1.0")
	c.RegisterType(c.FacadeServiceV1Descriptor, service1.NewFacadeServiceV1)
	return &c
}

```

```
package build

import (
	bbuild "github.com/pip-services-samples/client-beacons-gox/build"
	accbuild "github.com/pip-services-users/pip-clients-accounts-go/build"
	passbuild "github.com/pip-services-users/pip-clients-passwords-go/build"
	rolebuild "github.com/pip-services-users/pip-clients-roles-go/build"
	sessbuild "github.com/pip-services-users/pip-clients-sessions-go/build"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)

type ClientFacadeFactory struct {
	cbuild.CompositeFactory
}

func NewClientFacadeFactory() *ClientFacadeFactory {
	c := &ClientFacadeFactory{
		CompositeFactory: *cbuild.NewCompositeFactory(),
	}

	c.Add(accbuild.NewAccountsClientFactory())
	c.Add(sessbuild.NewSessionsClientFactory())
	c.Add(passbuild.NewPasswordsClientFactory())
	c.Add(rolebuild.NewRolesClientFactory())
	c.Add(bbuild.NewBeaconsClientFactory())
	return c
}

```

```
package container

import (
	ffactory "github.com/pip-services-samples/pip-samples-facade-go/build"
	cproc "github.com/pip-services4/pip-services4-go/pip-services4-container-go/container"
	httpbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	mbuild "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/build"
)

type FacadeProcess struct {
	*cproc.ProcessContainer
}

func NewFacadeProcess() *FacadeProcess {

	c := FacadeProcess{}
	c.ProcessContainer = cproc.NewProcessContainer("pip-facades-example", "Public facade for pip-vault 2.0")
	c.AddFactory(ffactory.NewClientFacadeFactory())
	c.AddFactory(ffactory.NewFacadeFactory())
	c.AddFactory(httpbuild.NewDefaultHttpFactory())
	c.AddFactory(mbuild.NewDefaultMongoDbFactory())

	return &c
}

```

```
package main

import (
	"context"
	"os"

	cont "github.com/pip-services-samples/pip-samples-facade-go/container"
)

func main() {
	proc := cont.NewFacadeProcess()
	proc.SetConfigPath("./config/config.yml")
	proc.Run(context.Background(), os.Args)
}

```

```
---
# Container info
- descriptor: "pip-services:container-info:default:default:*"
  name: "pip-facades-example"
  description: "Example Pip.Services facade"

# Console logger
- descriptor: "pip-services:logger:console:default:*"
  level: trace

# Log counters
- descriptor: "pip-services:counters:log:default:*"

# Mongodb connection
- descriptor: "pip-services:connection:mongodb:default:*"
  connection:
    uri: {{{MONGO_SERVICE_URI}}}
    host: {{{MONGO_SERVICE_HOST}}}{{#unless MONGO_SERVICE_HOST}}localhost{{/unless}}
    port: {{MONGO_SERVICE_PORT}}{{#unless MONGO_SERVICE_PORT}}27017{{/unless}}
    database: {{MONGO_DB}}{{#unless MONGO_DB}}app{{/unless}}
  credential:
    username: {{MONGO_USER}}
    password: {{MONGO_PASS}}


# Accounts components
# - descriptor: "pip-services-accounts:persistence:mongodb:default:*"
# - descriptor: "pip-services-accounts:controller:default:default:*"
# - descriptor: "pip-services-accounts:client:null:default:1.0"
- descriptor: "pip-services-accounts:client:memory:default:1.0"

# Roles components
# - descriptor: "pip-services-roles:persistence:mongodb:default:*"
# - descriptor: "pip-services-roles:controller:default:default:*"
# - descriptor: "pip-services-roles:client:memory:default:*"
- descriptor: "pip-services-roles:client:null:default:*"

# Passwords components
# - descriptor: "pip-services-passwords:persistence:mongodb:default:*"
# - descriptor: "pip-services-passwords:controller:default:default:*"
# - descriptor: "pip-services-passwords:client:memory:default:*"
- descriptor: "pip-services-passwords:client:null:default:*"

# Session components
# - descriptor: "pip-services-sessions:persistence:mongodb:default:*"
# - descriptor: "pip-services-sessions:controller:default:default:*"
# - descriptor: "pip-services-sessions:client:null:default:*"
- descriptor: "pip-services-sessions:client:memory:default:*"

# Beacons components
- descriptor: "beacons:persistence:mongodb:default:*"
- descriptor: "beacons:controller:default:default:*"
- descriptor: "beacons:client:direct:default:*"

# Main facade service
- descriptor: "pip-services:endpoint:http:default:*"
  root_path: ""
  connection:
    protocol: "http"
    host: "0.0.0.0"
    port: 8080

# Facade API V1
- descriptor: "pip-facades-example:controller:http:default:1.0"

# Hearbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"

# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"

```

```
version: '3.3'

services:

  app:
    image: ${IMAGE}
    environment:
      - MAGIC_CODE=magic
      - MONGO_SERVICE_URI=
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=app   
    ports:
      - "8080:8080"
    links:
      - mongo
      - accounts
      - role
      - passwords
      - sessions
      - beacons   
  
    command: node /app/bin/run.js -c /app/config/config-distributed.yml

  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"

  accounts:
    image: pipdevs/pip-services-accounts-node:latest
    environment:
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=accounts
    links:
      - mongo   
    ports:
      - "8082:8080"
  
  role:
    image: pipdevs/pip-services-roles-node:latest
    environment:
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=roles
    links:
        - mongo
    ports:
      - "8083:8080"

  passwords: 
    image: pipdevs/pip-services-passwords-node:1.0.1-12-rc
    environment:
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=passwd
    links:
        - mongo
    ports:
      - "8084:8080"

  sessions:
    image: pipdevs/pip-services-sessions-node:1.0.1-19-rc
    environment:
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=sessions
    links:
        - mongo
    ports:
      - "8085:8080"

  beacons:
    image: pipdevs/pip-services-beacons-node:1.0
    environment:
      - MONGO_SERVICE_HOST=mongo
      - MONGO_SERVICE_PORT=27017
      - MONGO_DB=beacons
    links:
        - mongo
    ports:
      - "8086:8080"
  

```

```
.\package.ps1

```

```
.\run.ps1

```

```

app_1    | [pip-facades-example:DEBUG:2021-06-17T22:03:56.128800] Opened REST service at http://0.0.0.0:8080
app_1    | [pip-facades-example:INFO:2021-06-17T22:03:56.180451] Container pip-facades-example started.
app_1    | [pip-facades-example:INFO:2021-06-17T22:03:56.180451] Press Control-C to stop the microservice...

```
