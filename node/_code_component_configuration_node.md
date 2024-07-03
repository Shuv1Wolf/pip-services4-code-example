```
interface IConfigurable {
	configure(config: ConfigParams): void;
}


```

```
import { ConfigParams } from "pip-services4-components-node"

let config = ConfigParams.fromTuples(
    "param1", 123,
    "param2", "2020-01-01T11:00:00.0Z"
);
```

```
let param1 = config.getAsInteger("param1");
let param2 = config.getAsDateTimeWithDefault("param2", new Date());


```

```
let configWithSections = ConfigParams.fromTuples(
    "param1", 123,
    "options.param1", "ABC",
    "options.param2", "XYZ"
);
let options = configWithSections.getSection("options");
```

```

let defaultConfig = ConfigParams.fromTuples(
  	"param1", 1,
  	"param2", "Default Value"
);
config = config.setDefaults(defaultConfig);


```

```
let anotherConfig = ConfigParams.fromString("param1=123;param2=ABC");


```

```
export class DataController implements IConfigurable {
    private _max_page_size: number = 5;
    public constructor() { }

    public configure(config: ConfigParams): void {
     this._max_page_size = config.getAsIntegerWithDefault('max_page_size', this._max_page_size);
    }

    public getData(ctx: Context, filter: FilterParams, paging: PagingParams): Promise<DataPage<MyData>> {
     paging.take = Math.min(paging.take, this._max_page_size);  
          // Get data using max page size constraint.
    }
}
```

```
let component = new DataController();
let config = ConfigParams.fromTuples("max_page_size", 100);
component.configure(config);



```

```
...
# Controller
- descriptor: "beacons:controller:default:default:1.0"
  max_page_size: 10
...


```

```
import { ConfigParams, NameResolver } from "pip-services4-components-node"

let config = ConfigParams.fromTuples(
	"descriptor", "myservice:connector:aws:connector1:1.0",
	"param1", "ABC",
	"param2", 123
);

let name = NameResolver.resolve(config); // Result: connector1

```

```
let config = ConfigParams.fromTuples(
	...
	"options.param1", "ABC",
	"options.param2", 123
);
let options = OptionsResolver.resolve(config); // Result: param1=ABC;param2=123


```

```
interface IConfigReader {
    readConfig(context: IContext, parameters: ConfigParams): Promise<ConfigParams>;
    addChangeListener(listener: INotifiable): void;
    removeChangeListener(listener: INotifiable): void;
}
```

```
let config = ConfigParams.fromTuples(
	"connection.host", "localhost",
	"connection.port", "8080"
);

let configReader = new MemoryConfigReader();
configReader.configure(config);

let parameters = ConfigParams.fromValue(process.env);
let result = await configReader.readConfig(ctx, parameters); 
// Result: connection.host=localhost;connection.port=8080
```

```
{ "key1": "{{KEY1_VALUE}}", "key2": "{{KEY2_VALUE}}" }

```

```
let configReader = new JsonConfigReader("config.json");
let parameters = ConfigParams.fromTuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC");
let result = await configReader.readConfig(ctx, parameters); // Result: key1=123;key2=ABC
```

```
key1: "1234"
key2: "ABCD"

```

```
let configReader = new YamlConfigReader("config.yml");
let parameters = ConfigParams.fromTuples("KEY1_VALUE", 123, "KEY2_VALUE", "ABC");
let result = await configReader.readConfig(ctx, parameters); // Result: key1=1234;key2=ABCD


```
