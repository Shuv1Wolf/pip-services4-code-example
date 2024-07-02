```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
connection:
  host: mongo
  port: 27017
```

```
import { ConfigParams, IConfigurable } from "pip-services4-components-node";
import { ConnectionParams } from "pip-services4-config-node";

class MyPersistence implements IConfigurable {
    private _host: string;
    private _port: number;

    public configure(config: ConfigParams) {
        let connection = ConnectionParams.fromConfig(config.getSection("connection"));
        this._host = connection.getHost();
        this._port = connection.getPortWithDefault(27017);
    }
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
import { 
    ConfigParams, IConfigurable, 
    IReferenceable, IReferences
} from "pip-services4-components-node";

import { CredentialResolver } from "pip-services4-config-node";


class MyPersistence implements IConfigurable, IReferenceable {
    ...
    private _credentialResolver = new CredentialResolver();
    private _username: string;
    private _password: string;

    public configure(config: ConfigParams) {
        ...
        this._credentialResolver.configure(config);
    }

    public setReferences(refs: IReferences) {
        ...
        this._credentialResolver.setReferences(refs);

        let credentials = this._credentialResolver.lookup(null);
        this._username = (await credentials).getUsername();
        this._password = (await credentials).getPassword();
    }
}

```

```
# MongoDb persistence component
descriptor: "myservice:mypersistance:mongodb:default:1.0"
credential:
  username: admin
  password: pass123

```

```
import { ConfigParams, IConfigurable } from "pip-services4-components-node";

import { CredentialParams } from "pip-services4-config-node";

class MyPersistence implements IConfigurable {
    ...
    private _username: string;
    private _password: string;

    public configure(config: ConfigParams) {
        ...
        let credentials = CredentialParams.fromConfig(config.getSection("credential"));
        this._username = credentials.getUsername();
        this._password = credentials.getPassword();
    }
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
import { 
    ConfigParams, IConfigurable, 
    IReferenceable, IReferences 
} from "pip-services4-components-node";

import { ConnectionResolver } from "pip-services4-config-node";

class MyPersistence implements IConfigurable, IReferenceable {
    private _connectionResolver = new ConnectionResolver();
    private _host: string;
    private _port: number;

    public configure(config: ConfigParams) {
        this._connectionResolver.configure(config);
    }

    public async setReferences(refs: IReferences) {
        this._connectionResolver.setReferences(refs);

        let connection = await this._connectionResolver.resolve(null);
        this._host = connection.getHost();
        this._port = connection.getPortWithDefault(27017);
    }
}
```
