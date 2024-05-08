```
import (
    auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)
```

```
import (
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)

// Case 1: Constructor + ConfigParams object
config := conf.NewConfigParamsFromTuples(
	"user", "user1",
	"password", "password1",
)
credential := auth.NewCredentialParams(config.Value()) // Returns {'credential.user': 'user1', 'credential.password': 'password1'}

// Case 2: Constructor + set/put methods
credential = auth.NewEmptyCredentialParams()
credential.SetUsername("user1")
credential.SetPassword("password1")
credential.SetStoreKey("store key1")
credential.SetAccessKey("access key 1")
credential.SetAccessId("access id 1")
credential.Put("parameter1", "value1")

fmt.Println(credential)
// Returns
//{'username': 'user1',
// 'password': 'password1',
// 'store_key': 'store key1',
// 'access_key': 'access key 1',
// 'access_id': 'access id 1'
// 'parameter1': 'value1'}
```

```
credential := auth.NewCredentialParamsFromTuples("username", "userA", "pin", "321", "credentialA", "a") // Returns {'username': 'userA', 'pin': '321', 'credentialA': 'a'}
```

```
credential := auth.NewCredentialParamsFromString("user=jdoe;pass=pass1") // Returns {'user': "'jdoe'", 'pass': "'pass1'"}
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)

config := conf.NewConfigParamsFromTuples("credential.user", "user1", "credential.password", "password1")
credential := auth.NewCredentialParamsFromConfig(config) // Returns {'user': 'user1', 'password': 'password1'}
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
	"credentials.database1.store_key", "store_key1",
	"credentials.database1.access_key", "access_key1",
	"credentials.database1.access_id", "access_id1",

	"credentials.database2.username", "user2",
	"credentials.database2.password", "userpass457",
	"credentials.database2.store_key", "store_key2",
	"credentials.database2.access_key", "access_key2",
	"credentials.database2.access_id", "access_id2",
	"credentials.database2.my_custom_credential_param", "myvalue",
)

credential := auth.NewManyCredentialParamsFromConfig(config)
// Returns [store_key=store_key2;
// access_key=access_key2;
// my_custom_credential_param=myvalue;
// access_id=access_id2;
// password=userpass457;
// username=user2
// access_key=access_key1;
// password=userpass123;
// username=user1;
// store_key=store_key1;
// access_id=access_id1]
```

```
credential[0].AddSection("sectionA", conf.NewConfigParamsFromTuples("key1", "ABCDE"))
// Returns [store_key=store_key2;
// access_key=access_key2;
// my_custom_credential_param=myvalue;
// access_id=access_id2;
// password=userpass457;
// username=user2
// access_key=access_key1;
// password=userpass123;
// username=user1;
// store_key=store_key1;
// access_id=access_id1;
// sectionA.key1=ABCDE]
```

```
credential[0].AccessId()  
credential[0].AccessKey() 
credential[0].Password()  
credential[0].Username()  
credential[0].StoreKey()  
```

```
credential[0].Get("username")
```

```
credential[0].SetPassword("password2")
```

```
credential[0].Put("password", "password3")
```

```
credential[0].SetAsObject("password", "password4")
```

```
config := conf.NewConfigParamsFromTuples("password", "password5")
overriden := credential[0].Override(config)
```

```
overriden[0].Remove("username")
```

```
import (
    auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)


config := conf.NewConfigParamsFromTuples(
	"key1.user", "jdoe",
	"key1.pass", "pass123",
	"key2.user", "bsmith",
	"key2.pass", "mypass",
)

credentialStore := auth.NewManyCredentialParamsFromConfig(config)
```

```
import (
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	auth "github.com/pip-services4/pip-services4-go/pip-services4-config-go/auth"
)


config := conf.NewConfigParamsFromTuples(
	"key1.user", "jdoe",
	"key1.pass", "pass123",
	"key2.user", "bsmith",
	"key2.pass", "mypass",
)

credentialStore := auth.NewEmptyMemoryCredentialStore()
credentialStore.ReadCredentials(config)
```

```
credential := auth.NewCredentialParams(conf.NewConfigParamsFromTuples(
	"user", "jdoe3", "password", "pass123345", "pin", "321345",
).Value())

credentialStore.Store(context.Background(), "key2", credential)
```

```
result, _ := credentialStore.Lookup(context.Background(), "key2")
fmt.Println(result) // Returns user=jdoe3;password=pass123345;pin=321345
```

```
credential1 := auth.NewCredentialParams(conf.NewConfigParamsFromTuples(
	"user", "jdoe3V2", "password", "pass123345", "pin", "321345",
).Value())

credentialStore.Store(context.Background(), "key3", credential1)
}
```

```
config = conf.NewConfigParamsFromTuples(
	"key1.user", "jdoeV2",
	"key1.pass", "pass123",
	"key4.user", "bsmith",
	"key4.pass", "mypass",
)

credentialStore.ReadCredentials(config)
```

```
credentialStore.Store(context.Background(), "key3", nil)
```
