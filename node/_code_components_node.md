```
import { 
    Parameters, ConfigParams, 
    Descriptor, ICleanable, IConfigurable, 
    IExecutable, INotifiable, IOpenable, 
    IReferenceable, IReferences, 
    IUnreferenceable, Context
} from "pip-services4-components-node";

class MyComponent implements IConfigurable, IReferenceable, IUnreferenceable, IOpenable, IExecutable, INotifiable, ICleanable {
    public constructor() { /* Initialize the component */ }
    public configure(config: ConfigParams) { /* configure the component */ }
    public setReferences(refs: IReferences) { /* set component dependencies */ }
    public unsetReferences() { /* unset component references */ }
    public isOpen(): boolean { /* return the component open state */ }
    public open(ctx: Context): Promise<void> { /* open the component */ }
    public close(ctx: Context): Promise<void> { /* close the component */ }
    public execute(ctx: Context, args: Parameters): Promise<any> { /* execute the component transaction */ }
    public notify(ctx: Context, args: Parameters) { /* notify the component about events */ }
    public clear(ctx: Context): Promise<void> { /* clear the component state */ }
}
```

```
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "pip-service-data"
  description: "Entities data microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Performance log counters
- descriptor: "pip-services:counters:log:default:1.0"

{{#if MONGO_ENABLED}}
# MongoDb persistence
- descriptor: "pip-service-data:persistence:mongodb:default:1.0"
  connection:
    uri: {{MONGO_SERVICE_URI}}
    host: {{MONGO_SERVICE_HOST}}{{#unless MONGO_SERVICE_HOST}}"localhost"{{/unless}}
    port: {{MONGO_SERVICE_PORT}}{{#unless MONGO_SERVICE_PORT}}27017{{/unless}}
    database: {{MONGO_DB}}{{#unless MONGO_DB}}"test"{{/unless}}
  credential:
    username: {{MONGO_USER}}
    password: {{MONGO_PASS}}
{{/if}}

{{#unless MONGO_ENABLED}}
# Default to in-memory persistence, if nothing is set
- descriptor: "pip-service-data:persistence:memory:default:1.0"
{{/unless}}

# Service
- descriptor: "pip-service-data:service:default:default:1.0"

{{#if HTTP_ENABLED}}
# Common HTTP endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{HTTP_PORT}}{{#unless HTTP_PORT}}8080{{/unless}}

# HTTP controller version 1.0
- descriptor: "pip-service-data:controller:http:default:1.0"
  swagger:
    enable: true

# Swagger service
- descriptor: "pip-services:swagger-controller:http:default:1.0"
{{/if}}

{{#if GRPC_ENABLED}}
# Common GRPC endpoint
- descriptor: "pip-services:endpoint:grpc:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: {{GRPC_PORT}}{{#unless GRPC_PORT}}8090{{/unless}}

# GRPC controller version 1.0
- descriptor: "pip-service-data:controller:grpc:default:1.0"
{{/if}}
```

```
import { ProcessContainer } from "pip-services4-container-node";
import { DefaultGrpcFactory } from "pip-services4-grpc-node";
import { DefaultHttpFactory } from "pip-services4-http-node";
import { DefaultSwaggerFactory } from "pip-services4-swagger-node";


class MyProcess extends ProcessContainer {
    public constructor() {
        super('mymicroservice', 'Sample microservice container');

        this._factories.add(new MyComponentFactory());
        this._factories.add(new DefaultHttpFactory());
        this._factories.add(new DefaultSwaggerFactory());
        this._factories.add(new DefaultGrpcFactory());
    }
}
```
