```
from pip_services4_config.auth import CredentialParams
```

```
# Case 1: Constructor + ConfigParams object
config = ConfigParams.from_tuples("user",  "user1", 
                                   "password", "password1")
credential = CredentialParams(config) # Returns {'credential.user': 'user1', 'credential.password': 'password1'}

# Case 2: Constructor + set/put methods
credential = CredentialParams()
credential.set_username("user1")
credential.set_password("password1")
credential.set_store_key("store key1")
credential.set_access_key("access key 1")
credential.set_access_id("access id 1")
credential.put("parameter1", "value1") 
credential
# Returns
#{'username': 'user1',
# 'password': 'password1',
# 'store_key': 'store key1',
# 'access_key': 'access key 1',
# 'access_id': 'access id 1'
# 'parameter1': 'value1'}

```

```
credential = CredentialParams.from_tuples("username","userA","pin", "321",'credentialA','a') # Returns {'username': 'userA', 'pin': '321', 'credentialA': 'a'}

```

```
credential = CredentialParams.from_string("user=jdoe;pass=pass1") # Returns {"user": "jdoe", "pass": "pass1"}

```

```
from pip_services4_config.auth import CredentialParams


config = ConfigParams.from_tuples("credential.user",  "user1", 
                                   "credential.password", "password1")
credential = CredentialParams.from_config(config) # Returns {'user': 'user1', 'password': 'password1'}

```

```
config = ConfigParams.from_tuples(
    # use connection if one connection and connections if more than one connection
    "connections.connection1.uri", "http://www.example1.com",
    "connections.connection1.protocol", "http",
    "connections.connection1.host", "host152",
    "connections.connection1.my_conn_parameter", "value1",
    "connections.connection2.uri", "http://www.example2.com",
    "connections.connection2.protocol", "http",
    "connections.connection2.host", "host153",
    "connections.connection2.my_conn_parameter", "value3",
    # use credential if one credential and credentials if more than one credential
    "credentials.database1.username", "user1",
    "credentials.database1.password", "userpass123",
    "credentials.database2.username", "user2",
    "credentials.database2.password", "userpass457",
    "credentials.database2.my_custom_credential_param", "myvalue"
)

credential = CredentialParams.many_from_config(config) 
# Returns [{'username': 'user1', 'password': 'userpass123'}, {'username': 'user2',  'password': 'userpass457','my_custom_credential_param': 'myvalue'}]

```

```
credential[0].add_section("sectionA", ConfigParams.from_tuples("key1", "ABCDE"))
# Returns
# {'username': 'user1',
# 'password': 'password1',
# 'store_key': 'store key1',
# 'access_key': 'access key 1',
# 'access_id': 'access id 1',
# 'sectionA.key1': 'ABCDE'}
```

```
credential[0].get_access_id()   # Returns 'access id 1'
credential[0].get_access_key()  # Returns 'access key 1'
credential[0].get_password()    # Returns 'password1'
credential[0].get_username()    # Returns 'user1'
credential[0].get_store_key()   # Returns 'store key1'

```

```
credential[0].get('username')   # Returns 'user1'
```

```
credential[0].set_password('password2')
```

```
credential[0].put('password', 'password3')
```

```
credential[0].set_as_object('password', 'password4')
```

```

config = ConfigParams.from_tuples("password", "password5")
overriden = credential[0].override(config)
# Returns
#{'username': 'user1',
# 'password': 'password5',
# 'store_key': 'store key1',
# 'access_key': 'access key 1',
# 'access_id': 'access id 1',
# 'parameter_name': 'new_parameter_value'}
```

```
overriden.remove('username')
```

```
from pip_services4_config.auth import MemoryCredentialStore
```

```
from pip_services4_config.auth import MemoryCredentialStore
from pip_services4_components.config import ConfigParams

config = ConfigParams.from_tuples("key1.user", "jdoe",
    "key1.pass", "pass123",
    "key2.user", "bsmith",
    "key2.pass", "mypass"
)

credentialStore = MemoryCredentialStore(config)
```

```
from pip_services4_config.auth import MemoryCredentialStore
from pip_services4_components.config import ConfigParams

config = ConfigParams.from_tuples("key1.user", "jdoe",
    "key1.pass", "pass123",
    "key2.user", "bsmith",
    "key2.pass", "mypass"
)

credentialStore = MemoryCredentialStore()
credentialStore.read_credentials(config)
```

```
credential = CredentialParams.from_tuples("user", "jdoe3", "password", "pass123345", "pin", "321345")
credentialStore.store(None, "key2", credential)
```

```
result = credentialStore.lookup("123", "key2")
result # Returns {'user': 'bsmith', 'pass': 'mypass'}
```

```
credential = CredentialParams.from_tuples("user", "jdoe3V2", "password", "pass123345", "pin", "321345")
credentialStore.store(None, "key3", credential)
```

```
config = ConfigParams.from_tuples(
    "key1.user", "jdoeV2",
    "key1.pass", "pass123",
    "key4.user", "bsmith",
    "key4.pass", "mypass"
)

credentialStore.read_credentials(config)
```

```

credentialStore.store(None, "key3", None)
```

```

```
