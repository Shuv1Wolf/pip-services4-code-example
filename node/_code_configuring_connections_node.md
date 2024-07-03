```
import { ConnectionParams } from "pip-services4-config-node"
```

```
import { ConnectionParams } from "pip-services4-config-node"
import { ConfigParams } from "pip-services4-components-node"

let config = ConfigParams.fromTuples("protocol", "http34343", "host", "host123", "uri", "uri321");
let connection = new ConnectionParams(config); // Returns {'protocol': 'http34343', 'host': 'host123', 'uri': 'uri321'}

```

```
let connection = new ConnectionParams();

connection.setDiscoveryKey("discovery key 1");
connection.setHost("localhost");
connection.setPort(8080);
connection.setProtocol("http");
connection.setUri("abc.com");

console.log(connection);  // Returns {'discovery_key': 'discovery key 1', 'host': 'localhost', 'port': '8080', 'protocol': 'http', 'uri': 'abc.com'}


```

```
connection.put("parameter_name", "paramter_value");
// connection: {'discovery_key': 'discovery key 1',
// 'host': 'localhost',
// 'port': '8080',
// 'protocol': 'http',
// 'uri': 'abc.com',
// 'parameter_name': 'paramter_value'}

```

```
let connection = ConnectionParams.fromTuples("protocol", "http", "host", "localhost", "port", "8080", "cluster", "mycluster");
// Returns {'protocol': 'http', 'host': 'localhost', 'port': '8080', 'cluster': 'mycluster'}

```

```
let connection = ConnectionParams.fromString("uri=abc2.com;protocol=http123"); // Returns {'uri': 'abc2.com', 'protocol': 'http123'}

```

```
let config = ConfigParams.fromTuples(
    "connection.protocol", "http34343", 
    "connection.host", "host123", 
    "connection.uri", "uri321"
);
let connection = ConnectionParams.fromConfig(config); // Returns {'protocol': 'http34343', 'host': 'host123', 'uri': 'uri321'}

```

```
let config = ConfigParams.fromTuples(
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
    "credentials.database2.my_custom_credential_param", "myvalue"
);

let connections = ConnectionParams.manyFromConfig(config);

// Returns a list with all connections

// [{'uri': 'http://www.example.com', 'protocol': 'http', 'host': 'host15', 'my_conn_parameter': 'value1'},
// {'uri': 'http://www.example2.com', 'protocol': 'http', 'host': 'host153', 'my_conn_parameter': 'value3'}]

```

```
connections[0].addSection("sectionA", ConfigParams.fromTuples("key1", "ABCDE"));
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
connections[0].getDiscoveryKey();                            // Returns 'discovery key 1'
connections[0].getHost();                                    // Returns 'localhost'
connections[0].getPort();                                    // Returns 8080
connections[0].getPortWithDefault(5050);                     // Returns 8080 or 0000 if port not defined
connections[0].getProtocol();                                // Returns 'http'
connections[0].getProtocolWithDefault("default protocol");   // Returns 'http' or 'default protocol' if protocol field not defined
connections[0].getUri();                                     // Returns 'abc.com'

```

```
connections[0].get("uri")
```

```
let section = connections[0].getSection("sectionA"); // Returns {'key1': 'ABCDE'}
```

```
connections[0].getSectionNames() 
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
connections[0].setHost("new host");
```

```
connections[0].put("host", "new host");
```

```
let overriden = connections[0].override(ConnectionParams.fromTuples("host", "new host"));
// Returns
// {'discovery_key': 'discovery key 1',
// 'host': 'new host',
// 'port': '8080',
// 'protocol': 'http',
// 'uri': 'abc.com'}
```

```
overriden.remove('uri');
```
