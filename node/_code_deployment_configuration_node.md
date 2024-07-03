```
import { FilterParams } from "pip-services4-data-node";

export interface IMyDataPersistence {
    getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend>;
    create(ctx: Context, item: MyFriend): Promise<MyFriend>;
}
```

```
import { ConfigParams, Descriptor, IConfigurable, IReferenceable, IReferences } from "pip-services4-components-node"
import { FilterParams } from "pip-services4-data-node";


export class HelloFriendController implements IConfigurable, IReferenceable {
    private defaultName: string;
    private persistence: IMyDataPersistence;

    public constructor() {
        this.defaultName = "Pip User";
    }

    public configure(config: ConfigParams): void {
        this.defaultName = config.getAsStringWithDefault("default_name", this.defaultName);
    }

    public setReferences(references: IReferences): void {
        this.persistence = references.getOneRequired(new Descriptor("hello-friend", "persistence", "*", "*", "1.0"));
    }

    public async greeting(): Promise<string> {
        let filter = FilterParams.fromTuples("type", "friend");
        let selectedFilter = await this.persistence.getOneRandom(null, filter);
        let name = selectedFilter != null ? selectedFilter.name : null;

        return `Hello, ${name} !`;
    }

    public async create(ctx: Context, item: MyFriend): Promise<MyFriend>  {
        let res = await this.persistence.create(ctx, item);
        return res;
    }
}
```

```
import { Context } from "pip-services4-components-node"
import { FilterParams } from "pip-services4-data-node";
import { IdentifiableMySqlPersistence } from "pip-services4-mysql-node";

export class HelloFriendPersistence1 extends IdentifiableMySqlPersistence<MyFriend, string> implements IMyDataPersistence {
    public constructor() {
        super("myfriends3");
    }

    protected defineSchema(): void {
        this.clearSchema();
        this.ensureSchema('CREATE TABLE IF NOT EXISTS `' + this._tableName + '` (id VARCHAR(32) PRIMARY KEY, `type` VARCHAR(50), `name` TEXT)');
    }

    private composeFilter(filter: FilterParams): string {
        filter ??= new FilterParams();
        let type = filter.getAsNullableString("type");
        let name = filter.getAsNullableString("name");

        let filterCondition = "";
        if (type != null)
            filterCondition += "type='" + type + "'";
        if (name != null)
            filterCondition += "name='" + name + "'";

        return filterCondition;
    }

    public getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend> {
        return super.getOneRandom(ctx, this.composeFilter(filter));
    }
}
```

```
import { Context } from "pip-services4-components-node"
import { FilterParams } from "pip-services4-data-node";
import { IdentifiablePostgresPersistence } from "pip-services4-postgres-node";

export class HelloFriendPersistence2 extends IdentifiablePostgresPersistence<MyFriend, string> implements IMyDataPersistence {
    public constructor() {
        super("myfriends3");
    }

    protected defineSchema(): void {
        this.clearSchema();
        this.ensureSchema('CREATE TABLE IF NOT EXISTS ' + this._tableName + ' (id TEXT PRIMARY KEY, type TEXT, name TEXT)');
    }

    private composeFilter(filter: FilterParams): string {
        filter ??= new FilterParams();
        let type = filter.getAsNullableString("type");
        let content = filter.getAsNullableString("content");

        let filterCondition = "";
        if (type != null)
            filterCondition += "type='" + type + "'";
        if (content != null)
            filterCondition += "content='" + content + "'";

        return filterCondition;
    }

    public getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend> {
        return super.getOneRandom(ctx, this.composeFilter(filter));
    }
}
```

```
---
# Container context
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "hello-friend"
  description: "HelloFriend microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0"
  level: "trace"

# Performance counter that post values to log
- descriptor: "pip-services:counters:log:default:1.0"

# Serivce
- descriptor: "hello-friend:service:default:default:1.0"
  default_name: "Friend"

# Shared HTTP Endpoint
- descriptor: "pip-services:endpoint:http:default:1.0"
  connection:
    protocol: http
    host: 0.0.0.0
    port: 8080

# HTTP Controller V1
- descriptor: "hello-friend:controller:http:default:1.0"

# Heartbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"

# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"


{{#if MYSQL_ENABLED}}
# Persistence - MySQL
- descriptor: "hello-friend:persistence:mysql:default:1.0"
  connection:
    host: 'localhost'
    port: '3306'
    database: 'pip'
  credential:
    username: 'root'
    password: ''
{{/if}}

{{#if POSTGRES_ENABLED}}
# Persistence - PostgreSQL
- descriptor: "hello-friend:persistence:postgres:default:1.0"
  connection:
    host: 'localhost'
    port: '5432'
    database: 'pip'
  credential:
    username: 'postgres'
    password: 'admin'
{{/if}}
```

```
import { Context, Descriptor, Factory, ConfigParams, IReferences, IConfigurable, IReferenceable, IContext } from "pip-services4-components-node"
import { ProcessContainer } from "pip-services4-container-node";
import { FilterParams,IStringIdentifiable } from "pip-services4-data-node";
import { DefaultHttpFactory, RestController } from "pip-services4-http-node";
import { IdentifiablePostgresPersistence } from "pip-services4-postgres-node";
import { IdentifiableMySqlPersistence } from "pip-services4-mysql-node";

export class MyFriend implements IStringIdentifiable {
    public id: string;
    public type: string;
    public name: string;
}

export class HelloFriendRestController extends RestController {
    protected service: HelloFriendService;

    public constructor() {
        super()
        this._baseRoute = "/hello_friend";
    }

    public configure(config: ConfigParams): void {
        super.configure(config);
    }

    public setReferences(references: IReferences): void {
        super.setReferences(references);
        this.service = references.getOneRequired(new Descriptor("hello-friend", "service", "*", "*", "1.0"));
    }

    private async greeting(req: any, res: any): Promise<void> {
        let result = await this.service.greeting();
        await this.sendResult(req, res, result);
    }

    private async create(req: any, res: any): Promise<void> {
        let correlationId = this.getTraceId(req);
        let friend: MyFriend = req.query;
        let result = await this.service.create(correlationId, friend);

        await this.sendResult(req, res, result);
    }

    public register(): void {
        this.registerRoute("GET", "/greeting", null, this.greeting);
        this.registerRoute("GET", "/greeting_create", null, this.create);
    }
}



export class HelloFriendService implements IConfigurable, IReferenceable {
    private defaultName: string;
    private persistence: IMyDataPersistence;

    public constructor() {
        this.defaultName = "Pip User";
    }

    public configure(config: ConfigParams): void {
        this.defaultName = config.getAsStringWithDefault("default_name", this.defaultName);
    }

    public setReferences(references: IReferences): void {
        this.persistence = references.getOneRequired(new Descriptor("hello-friend", "persistence", "*", "*", "1.0"));
    }

    public async greeting(): Promise<string> {
        let filter = FilterParams.fromTuples("type", "friend");
        let selectedFilter = await this.persistence.getOneRandom(null, filter);
        let name = selectedFilter != null ? selectedFilter.name : null;

        return `Hello, ${name} !`;
    }

    public async create(ctx: Context, item: MyFriend): Promise<MyFriend>  {
        let res = await this.persistence.create(ctx, item);
        return res;
    }
}

  
  
export interface IMyDataPersistence {
    getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend>;
    create(ctx: Context, item: MyFriend): Promise<MyFriend>;
}
    


export class HelloFriendPersistence1 extends IdentifiableMySqlPersistence<MyFriend, string> implements IMyDataPersistence {
    public constructor() {
        super("myfriends3");
    }
    get(key: string) {
        throw new Error("Method not implemented.");
    }

    protected defineSchema(): void {
        this.clearSchema();
        this.ensureSchema('CREATE TABLE IF NOT EXISTS `' + this._tableName + '` (id VARCHAR(32) PRIMARY KEY, `type` VARCHAR(50), `name` TEXT)');
    }

    private composeFilter(filter: FilterParams): string {
        filter ??= new FilterParams();
        let type = filter.getAsNullableString("type");
        let name = filter.getAsNullableString("name");

        let filterCondition = "";
        if (type != null)
            filterCondition += "type='" + type + "'";
        if (name != null)
            filterCondition += "name='" + name + "'";

        return filterCondition;
    }

    public getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend> {
        return super.getOneRandom(ctx, this.composeFilter(filter));
    }
}


   
export class HelloFriendPersistence2 extends IdentifiablePostgresPersistence<MyFriend, string> implements IMyDataPersistence {
    public constructor() {
        super("myfriends3");
    }

    protected defineSchema(): void {
        this.clearSchema();
        this.ensureSchema('CREATE TABLE IF NOT EXISTS ' + this._tableName + ' (id TEXT PRIMARY KEY, type TEXT, name TEXT)');
    }

    private composeFilter(filter: FilterParams): string {
        filter ??= new FilterParams();
        let type = filter.getAsNullableString("type");
        let content = filter.getAsNullableString("content");

        let filterCondition = "";
        if (type != null)
            filterCondition += "type='" + type + "'";
        if (content != null)
            filterCondition += "content='" + content + "'";

        return filterCondition;
    }

    public getOneRandom(ctx: Context, filter: FilterParams): Promise<MyFriend> {
        return super.getOneRandom(ctx, this.composeFilter(filter));
    }
}



export class HelloFriendControllerFactory extends Factory {
    public constructor() {
        super();
        let HttpControllerDescriptor = new Descriptor("hello-friend", "controller", "http", "*", "1.0");      // View
        let ServiceDescriptor = new Descriptor("hello-friend", "service", "default", "*", "1.0"); // service
        let PersistenceDescriptor1 = new Descriptor("hello-friend", "persistence", "mysql", "*", "1.0"); // Persistence
        let PersistenceDescriptor2 = new Descriptor("hello-friend", "persistence", "postgres", "*", "1.0"); // Persistence

        this.registerAsType(HttpControllerDescriptor, HelloFriendRestController); // View
        this.registerAsType(ServiceDescriptor, HelloFriendService);  // Service
        this.registerAsType(PersistenceDescriptor1, HelloFriendPersistence1); // Persistence
        this.registerAsType(PersistenceDescriptor2, HelloFriendPersistence2); // Persistence
    }
}



export class HelloFriendProcess extends ProcessContainer {
    public constructor() {
        super("hello-friend", "HelloFriend microservice");
        this._configPath = "./config.yaml"
        this._factories.add(new HelloFriendControllerFactory());
        this._factories.add(new DefaultHttpFactory());
    }
}



export async function main() { 
    try {
        // Step 1 - Database selection
        // process.env['MYSQL_ENABLED'] = 'true';
        process.env['POSTGRES_ENABLED'] = 'true';

        // Step 2 - The run() command
        let proc = new HelloFriendProcess();
        proc.run(process.argv);
    } catch (ex) {
        console.error(ex);
    }
}

```

```
export async function main() { 
    try {
        // Step 1 - Database selection
        // process.env['MYSQL_ENABLED'] = 'true';
        process.env['POSTGRES_ENABLED'] = 'true';

        // Step 2 - The run() command
        let proc = new HelloFriendProcess();
        proc.run(process.argv);
    } catch (ex) {
        console.error(ex);
    }
}
```

```

```
