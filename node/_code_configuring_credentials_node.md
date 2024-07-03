```
import { CredentialParams } from "pip-services4-config-node"
```

```
import { CredentialParams } from "pip-services4-config-node"
import { ConfigParams } from "pip-services4-components-node"

// Case 1: Constructor + ConfigParams object
let config = ConfigParams.fromTuples("user", "user1", "password", "password1");
let credential = new CredentialParams(config); // Returns {'credential.user': 'user1', 'credential.password': 'password1'}

// Case 2: Constructor + set/put methods
credential = new CredentialParams();
credential.setUsername("user1");
credential.setPassword("password1");
credential.setStoreKey("store key1");
credential.setAccessKey("access key 1");
credential.setAccessId("access id 1");
credential.put("parameter1", "value1");

credential
// Returns
// {'username': 'user1',
// 'password': 'password1',
// 'store_key': 'store key1',
// 'access_key': 'access key 1',
// 'access_id': 'access id 1'
// 'parameter1': 'value1'}
```

```
let credential = CredentialParams.fromTuples("username", "userA", "pin", "321", "credentialA", "a");
```

```
let credential = CredentialParams.fromString("user=jdoe;pass=pass1"); // Returns {"user": "jdoe", "pass": "pass1"}

```

```
import { CredentialParams } from "pip-services4-config-node"
import { ConfigParams } from "pip-services4-components-node"

let config = ConfigParams.fromTuples("credential.user", "user1", "credential.password", "password1");
let credential = CredentialParams.fromConfig(config); // Returns {'user': 'user1', 'password': 'password1'}

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

let credential = CredentialParams.manyFromConfig(config);

// Returns [{'username': 'user1', 'password': 'userpass123'}, {'username': 'user2',  'password': 'userpass457','my_custom_credential_param': 'myvalue'}]

```

```
credential[0].addSection("sectionA", ConfigParams.fromTuples("key1", "ABCDE"));
// Returns
// {'username': 'user1',
// 'password': 'password1',
// 'store_key': 'store key1',
// 'access_key': 'access key 1',
// 'access_id': 'access id 1',
// 'sectionA.key1': 'ABCDE'}
```

```
credential[0].getAccessId();   // Returns 'access id 1'
credential[0].getAccessKey();  // Returns 'access key 1'
credential[0].getPassword();   // Returns 'password1'
credential[0].getUsername();   // Returns 'user1'
credential[0].getStoreKey();   // Returns 'store key1'
```

```
credential[0].get("username");   // Returns 'user1'
```

```
credential[0].setPassword("password2");
```

```
credential[0].put("password", "password3");

```

```
credential[0].SetAsObject("password", "password4");

```

```
let config = ConfigParams.fromTuples("password", "password5");
let overriden = credential[0].override(config);

// Returns
// {'username': 'user1',
// 'password': 'password5',
// 'store_key': 'store key1',
// 'access_key': 'access key 1',
// 'access_id': 'access id 1',
// 'parameter_name': 'new_parameter_value'}

```

```
overriden.remove("username");

```

```
import { MemoryCredentialStore } from "pip-services4-config-node"
```

```
import { MemoryCredentialStore } from "pip-services4-config-node"
import { ConfigParams } from "pip-services4-components-node"

let config = ConfigParams.fromTuples(
    "key1.user", "jdoe",
    "key1.pass", "pass123",
    "key2.user", "bsmith",
    "key2.pass", "mypass"
);
let credentialStore = new MemoryCredentialStore(config);
```

```
import { MemoryCredentialStore } from "pip-services4-config-node"
import { ConfigParams } from "pip-services4-components-node"

let config = ConfigParams.fromTuples(
    "key1.user", "jdoe",
    "key1.pass", "pass123",
    "key2.user", "bsmith",
    "key2.pass", "mypass"
);

let credentialStore = new MemoryCredentialStore();
credentialStore.readCredentials(config);
```

```
let credential = new CredentialParams(ConfigParams.fromTuples("user", "jdoe3", "password", "pass123345", "pin", "321345"));
await credentialStore.store(ctx, "key2", credential);

```

```
let result = await credentialStore.lookup(ctx, "key2");
console.log(result); // Returns {'user': 'bsmith', 'pass': 'mypass'}
```

```
let credential1 = new CredentialParams(ConfigParams.fromTuples("user", "jdoe3V2", "password", "pass123345", "pin", "321345"));
await credentialStore.store(ctx, "key3", credential1);
```

```
config = ConfigParams.fromTuples(
    "key1.user", "jdoeV2",
    "key1.pass", "pass123",
    "key4.user", "bsmith",
    "key4.pass", "mypass"
);
credentialStore.readCredentials(config);
```

```
await credentialStore.store(ctx, "key3", null);
```
