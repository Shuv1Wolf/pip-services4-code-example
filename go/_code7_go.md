### Connection parameters

```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
connection:
  host: mongo
  port: 27017

```

```

import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cconn "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)

type MyPersistence struct {
	host string
	port int
}

func (c *MyPersistence) Configure(ctx context.Context, config *cconf.ConfigParams) {
	connection := cconn.NewConnectionParamsFromConfig(config.GetSection("connection"))
	c.host = connection.Host()
	c.port = connection.Port()
}
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
import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cauth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)

type MyPersistence struct {
	credentialResolver *cauth.CredentialResolver
	username           string
	password           string
}

func (c *MyPersistence) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// ...
	c.credentialResolver.Configure(ctx, config)
}

func (c *MyPersistence) SetReferences(ctx context.Context, references cref.IReferences) {
	// ...
	c.credentialResolver.SetReferences(ctx, references)

	credentials, err := c.credentialResolver.Lookup(ctx)
	c.username = credentials.Username()
	c.password = credentials.Password()
}

```

### Credential parameters

```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
credential:
  username: admin
  password: pass123

```

```
import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cauth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)

type MyPersistence struct {
	// ...
	username string
	password string
}

func (c *MyPersistence) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// ...
	credentials := cauth.NewCredentialParamsFromConfig(config)
	c.username = credentials.Username()
	c.password = credentials.Password()
}

func (c *MyPersistence) SetReferences(ctx context.Context, references cref.IReferences) {
	// ...
}
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
import (
	"context"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cconn "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)

type MyPersistence struct {
	// ...
	connectionResolver *cconn.ConnectionResolver
	host               string
	port               int
}

func (c *MyPersistence) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// ...
	c.connectionResolver.Configure(ctx, config)
}

func (c *MyPersistence) SetReferences(ctx context.Context, references cref.IReferences) {
	c.connectionResolver.SetReferences(ctx, references)
	connection, err := c.connectionResolver.Resolve(ctx)
	c.host = connection.Host()
	c.port = connection.Port()
}
```
