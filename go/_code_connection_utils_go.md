```
import (
	ccon "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)
```

```
import (
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	ccon "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)


options := cconf.NewConfigParamsFromTuples(
	"host", "broker1,broker2",
	"port", ",8082",
	"username", "user",
	"password", "pass123",
	"param1", "ABC",
	"param2", "XYZ",
	"param3", nil,
)

uri := ccon.ConnectionUtils.ComposeUri(options, "tcp", 9092)
```

```
configA := cconf.NewConfigParamsFromTuples(
	"host", "broker1,broker2",
	"port", "8082",
	"username", "user2",
	"password", "pass123",
	"param1", "ABC",
	"param2", "XYZ",
	"param3", "p3A",
)

configB := cconf.NewConfigParamsFromTuples(
	"host", "broker3,broke42",
	"port", "8083",
	"username", "user2",
	"password", "pass1234",
	"param1", "ABCD",
	"param2", "WXYZ",
	"param3", "p3B",
)

config := ccon.ConnectionUtils.Concat(configA, configB, "username", "password")
```

```

config := cconf.NewConfigParamsFromTuples(
	"host", "broker1,broker2",
	"port", ",8082",
	"username", "user",
	"password", "pass123",
	"param1", "ABC",
	"param2", "XYZ",
	"param3", nil,
)

config2 := ccon.ConnectionUtils.Exclude(config, "username", "password")
```

```
config := cconf.NewConfigParamsFromTuples(
	"host", "broker1,broker2",
	"port", ",8082",
	"username", "user",
	"password", "pass123",
	"param1", "ABC",
	"param2", "XYZ",
	"param3", nil,
)

config2 := ccon.ConnectionUtils.Include(config, "username", "password")
```

```
config := ccon.ConnectionUtils.ParseUri("tcp://user:pass123@broker1:8082,broker2:8082?param1=ABC&param2=XYZ", "tcp", 9092)

```

```
config := ccon.ConnectionUtils.ParseUri("user:pass123@broker1,broker2?param1=ABC&param2=XYZ", "tcp", 9092)
```

```
import (
	"context"
	"fmt"

	cerror "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/errors"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	ccon "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
	mongodrv "go.mongodb.org/mongo-driver/mongo"
	mongoclopt "go.mongodb.org/mongo-driver/mongo/options"
	"go.mongodb.org/mongo-driver/x/mongo/driver/connstring"
)

func main() {
	options := cconf.NewConfigParamsFromTuples(
		"host", "localhost",
		"port", ",27017",
		"username", "user",
		"password", "pass123",
		"protocol", "mongodb",
		"collection", "my_db_name",
	)

	conn := NewMongoDbConnector(false)
	conn.Configure(context.Background(), options)
	err := conn.Open(context.Background(), "123")

	fmt.Println(err)
}

type MongoDbConnector struct {
	//   The configuration options.
	Config *cconf.ConfigParams
	//   The MongoDB connection object.
	Connection *mongodrv.Client
	//   The MongoDB database name.
	DatabaseName string
	//   The MongoDb database object.
	Db *mongodrv.Database

	secureMongo bool
}

// NewMongoDbConnector are creates a new instance of the connection component.
// Returns *MongoDbConnector with default config
func NewMongoDbConnector(secureMongo bool) *MongoDbConnector {
	c := MongoDbConnector{
		// The configuration options.
		secureMongo: secureMongo,
		Config:      cconf.NewEmptyConfigParams(),
	}
	return &c
}

func (c *MongoDbConnector) GetCollection() *mongodrv.Collection {
	return c.Db.Collection("test")
}

func (c *MongoDbConnector) Configure(ctx context.Context, config *cconf.ConfigParams) {
	c.Config = config

	_, ok := c.Config.GetAsNullableString("uri")
	if !ok {
		c.Config = ccon.ConnectionUtils.ParseUri(c.Config.GetAsString("uri"), "mongodb", 27017)
	}

	// if mongo without auth
	if !c.secureMongo {
		c.Config = ccon.ConnectionUtils.Exclude(c.Config, "username", "password")
	}
}

func (c *MongoDbConnector) IsOpen() bool {
	return c.Connection != nil
}

func (c *MongoDbConnector) composeSettings(settings *mongoclopt.ClientOptions) {
	authUser := c.Config.GetAsString("auth_user")
	authPassword := c.Config.GetAsString("auth_password")

	// Auth params
	if authUser != "" && authPassword != "" {
		authParams := mongoclopt.Credential{
			Username: authUser,
			Password: authPassword,
		}
		settings.SetAuth(authParams)
	}
}

func (c *MongoDbConnector) Open(ctx context.Context, correlationId string) error {
	collection, _ := c.Config.GetAsNullableString("collection")

	c.Config = ccon.ConnectionUtils.Exclude(c.Config, "collection")

	uri := ccon.ConnectionUtils.ComposeUri(c.Config, "mongodb", 27017)
	uri += "/" + collection

	settings := mongoclopt.Client()
	settings.ApplyURI(uri)
	c.composeSettings(settings)

	client, err := mongodrv.NewClient(settings)

	if err != nil {
		err = cerror.NewConnectionError(correlationId, "CONNECT_FAILED", "Create client for mongodb failed").WithCause(err)
		return err
	}
	cs, _ := connstring.Parse(uri)
	c.DatabaseName = cs.Database

	err = client.Connect(ctx)
	if err != nil {
		err = cerror.NewConnectionError(correlationId, "CONNECT_FAILED", "Connection to mongodb failed").WithCause(err)
		return err
	}
	c.Connection = client
	c.Db = client.Database(c.DatabaseName)
	return nil
}

func (c *MongoDbConnector) Close(ctx context.Context, correlationId string) error {
	if c.Connection == nil {
		return nil
	}

	err := c.Connection.Disconnect(ctx)
	c.Connection = nil
	c.Db = nil
	c.DatabaseName = ""

	if err != nil {
		err = cerror.NewConnectionError(correlationId, "DISCONNECT_FAILED", "Disconnect from mongodb failed: ").WithCause(err)
	}

	return err
}

```
