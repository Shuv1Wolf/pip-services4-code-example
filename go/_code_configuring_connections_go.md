```
import (
	conn "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)
```

```
import (
	conn "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)

config := conn.NewConnectionParamsFromTuples(
	"protocol", "http34343", "host", "host123", "uri", "uri321",
)

connection := conn.NewConnectionParams(config.Value()) // Returns {'protocol': 'http34343', 'host': 'host123', 'uri': 'uri321'}

```

```
connection := conn.NewEmptyConnectionParams()

connection.SetDiscoveryKey("discovery key 1")
connection.SetHost("localhost")
connection.SetPort(8080)
connection.SetProtocol("http")
connection.SetUri("abc.com")

fmt.Println(connection)  // Returns {'discovery_key': 'discovery key 1', 'host': 'localhost', 'port': '8080', 'protocol': 'http', 'uri': 'abc.com'}
```

```
connection.Put("parameter_name", "paramter_value")
// connection: {'discovery_key': 'discovery key 1',
// 'host': 'localhost',
// 'port': '8080',
// 'protocol': 'http',
// 'uri': 'abc.com',
// 'parameter_name': 'paramter_value'}
```

```
connection := conn.NewConnectionParamsFromTuples(
	"protocol", "http", 
	"host", "localhost", 
	"port", "8080", 
	"cluster", "mycluster",
)
// Returns {'protocol': 'http', 'host': 'localhost', 'port': '8080', 'cluster': 'mycluster'}
```

```
connection := conn.NewConnectionParamsFromString("uri=abc2.com;protocol=http123") // Returns {'uri': 'abc2.com', 'protocol': 'http123'}
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	conn "github.com/pip-services4/pip-services4-go/pip-services4-config-go/connect"
)


config := conf.NewConfigParamsFromTuples(
	"connection.protocol", "http34343",
	"connection.host", "host123",
	"connection.uri", "uri321",
)
connection := conn.NewConnectionParamsFromConfig(config) // Returns {'protocol': 'http34343', 'host': 'host123', 'uri': 'uri321'}

```

```
config := conf.NewConfigParamsFromTuples(
	// use connection if one connection and connections if more than one connection
	"connections.connection1.uri", "http://www.example1.com",
	"connections.connection1.protocol", "http",
	"connections.connection1.host", "host152",
	"connections.connection1.my_conn_parameter", "value1",
	"connections.connection2.uri", "http://www.example2.com",
	"connections.connection2.protocol", "http",
	"connections.connection2.host", "host153",
	"connections.connection2.my_conn_parameter", "value3",
	// use credential if one credential and credentials if more than one credential
	"credentials.database1.username", "user1",
	"credentials.database1.password", "userpass123",
	"credentials.database2.username", "user2",
	"credentials.database2.password", "userpass457",
	"credentials.database2.my_custom_credential_param", "myvalue",
)

connection := conn.NewManyConnectionParamsFromConfig(config)

// Returns a list with all connections

// [{'uri': 'http://www.example.com', 'protocol': 'http', 'host': 'host15', 'my_conn_parameter': 'value1'},
// {'uri': 'http://www.example2.com', 'protocol': 'http', 'host': 'host153', 'my_conn_parameter': 'value3'}]
```

```
connection.AddSection("sectionA", conf.NewConfigParamsFromTuples("key1", "ABCDE"))
// connection: 
// {'discovery_key': 'discovery key 1',
// 'host': 'localhost',
// 'port': '8080',
// 'protocol': 'http',
// 'uri': 'abc.com',
// 'parameter_name': 'paramter_value',
// 'sectionA.key1': 'ABCDE'}
```

```
connection.DiscoveryKey()                          // Returns "discovery key 1"
connection.Host()                                  // Returns "localhost"
connection.Port()                                  // Returns 8080
connection.PortWithDefault(5050)                   // Returns 8080 or 0000 if port not defined
connection.Protocol()                              // Returns "http"
connection.ProtocolWithDefault("default protocol") // Returns "http" or "default protocol" if protocol field not defined
connection.Uri()                                   // Returns "abc.com"
```

````
connection.Get("uri")
````

```
section := connection.GetSection("sectionA") // Returns {'key1': 'ABCDE'}
```

```
connection.GetSectionNames()
// Returns
// ['discovery_key',
// 'host',
// 'port',
// 'protocol',
// 'uri',
// 'parameter_name',
// 'sectionA']
```

```
connection.SetHost("new host")
```

```
connection.Put("host", "new host")
```

```
overriden := connection.Override(conf.NewConfigParamsFromTuples("host", "new host"))
// Returns
// {'discovery_key': 'discovery key 1',
// 'host': 'new host',
// 'port': '8080',
// 'protocol': 'http',
// 'uri': 'abc.com'}
```

```
overriden.Remove("uri")
```
