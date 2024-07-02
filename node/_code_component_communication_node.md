```
import { References } from "pip-services4-components-node";
```

```
import { ConfigParams } from "pip-services4-components-node";
import { PrometheusMetricsController } from "pip-services4-prometheus-node";

let controller = new PrometheusMetricsController();

controller.configure(ConfigParams.fromTuples(
    "connection.protocol", "http",
    "connection.host", "localhost",
    "connection.port", 8080
));
```

```
import { Descriptor, References } from "pip-services4-components-node";

let references = References.fromTuples(
  new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller
);

```

```
let references2 = new References();
```

```
let references3 = new References([new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller]);
```

```
references.find(controller, true);
```

```
references.getAll();
```

```
references.getAllLocators();
```

```
references.getOneOptional(controller);
```

```
references.getOneRequired(controller);
```

```
references.getOptional(controller);
```

```
references.getRequired(controller);
```

```
references2.put(new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller);
```

```
references2.remove(controller);
```

```
references.removeAll(controller);
```

```
let context_info = new ContextInfo();
context_info.name = "Test";
context_info.description = "This is a test container";

let references = References.fromTuples(
    new Descriptor("pip-services", "context-info", "default", "default", "1.0"), context_info,
    new Descriptor("pip-services", "counters", "prometheus", "default", "1.0"), counters,
    new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller
);

controller.setReferences(references);
counters.setReferences(references);
```

```
class MyComponentA implements IReferenceable {
    public consoleLog: boolean = true; // console log flag
    private counters: CachedCounters;

    public constructor() {

        if (this.consoleLog)
            console.log("MyComponentA has been created.");
    }
    public setReferences(references: IReferences): void {
        this.counters = references.getOneRequired<CachedCounters>(
            new Descriptor("*", "counters", "*", "*", "*")
        );
    }
}

```

```
/**
	 * Sets references to dependent components.
	 * 
	 * @param references 	references to locate the component dependencies. 
     */
    public setReferences(references: IReferences): void {
        this._logger.setReferences(references);
        this._connectionResolver.setReferences(references);

        let contextInfo = references.getOneOptional<ContextInfo>(
            new Descriptor("pip-services", "context-info", "default", "*", "1.0"));
        if (contextInfo != null && this._source == null) {
            this._source = contextInfo.name;
        }
        if (contextInfo != null && this._instance == null) {
            this._instance = contextInfo.contextId;
        }
    }
```

```
 /**
	 * Sets references to dependent components.
	 * 
	 * @param references 	references to locate the component dependencies. 
     */
    public setReferences(references: IReferences): void {
        super.setReferences(references);

        this._cachedCounters = this._dependencyResolver.getOneOptional<PrometheusCounters>("prometheus-counters");
        if (this._cachedCounters == null)
            this._cachedCounters = this._dependencyResolver.getOneOptional<CachedCounters>("cached-counters");

        let contextInfo = references.getOneOptional<ContextInfo>(
            new Descriptor("pip-services", "context-info", "default", "*", "1.0"));

        if (contextInfo != null && (this._source == "" || this._source == undefined)) {
            this._source = contextInfo.name;
        }
        if (contextInfo != null && (this._instance == "" || this._instance == undefined)) {
            this._instance = contextInfo.contextId;
        }
    }
```

```
let references = References.fromTuples(
    new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "1.0"), controller,
    new Descriptor("pip-services", "metrics-controller", "prometheus", "default", "2.0"), controller2,
    new Descriptor("pip-services", "counters", "prometheus", "default", "1.0"), counters,
    new Descriptor("pip-services", "context-info", "default", "*", "1.0"), new ContextInfo("tutorial", "Example context conmponent"),
    new Descriptor("pip-services", "logger", "console", "default", "1.0"), new ConsoleLogger()
);
```

```
references.find(new Descriptor("*", "metrics-controller", "*", "*", "*"), true);

```

```
references.getOneOptional(new Descriptor("*", "metrics-controller", "*", "*", "2.0"));

```

```
references.getRequired(new Descriptor("pip-services", "*", "*", "*", "*"));

```

```
let reference = references.getOneOptional(controller);

```
