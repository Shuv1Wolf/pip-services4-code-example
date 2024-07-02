```
interface IReferences {
	put(locator: any, component: any);
	remove(locator: any): any;
	removeAll(locator: any): any[];
	getAllLocators(): any[];
	getAll(): any[];
	getOptional<T>(locator: any): T[];
	getRequired<T>(locator: any): T[];
	getOneOptional<T>(locator: any): T;
	getOneRequired<T>(locator: any): T;
	find<T>(locator: any, required: boolean): T[];
}


```

```
interface IUnreferenceable {
 	unsetReferences(): void;
}


```

```
class Worker1 {
    private _defaultName: string;

    constructor(name) {
        this._defaultName = name || "Default name1";
    }

    public do(level, message) {
        console.log('Write to ${this._defaultName}.${level} message: ${message}');
    }
}

class Worker2 {
    private _defaultName: string;
  
    constructor(name) {
        this._defaultName = name || "Default name2";
    }

    public do(level, message) {
        console.log('Write to ${this._defaultName}.${level} message: ${message}');
    }
}

```

```
import { IReferenceable, IUnreferenceable } from "pip-services4-components-node";
import { LogLevel } from "pip-services4-observability-node";

class SimpleController implements IReferenceable, IUnreferenceable {
  private _worker: any;
  
  constructor() {}

  public setReferences(references) {
      this._worker = references.getOneRequired(111)
  }
  public unsetReferences() {
      this._worker = null;
  }
  public greeting(name) {
      this._worker.do(LogLevel.Debug,  "Hello, " + (name) + "!");
  }
}
```

```
let references = References.fromTuples(
  111, new Worker1(null),
  222, new Worker2(null)
);

let controller = new SimpleController();
controller.setReferences(references);
controller.greeting("world");
controller.unsetReferences();
controller = null;
```

```
class Descriptor {
	private _group: string;
	private _type: string;
	private _kind: string;
	private _name: string;
	private _version: string;
	public getGroup(): string;
	public getType(): string;
	public getKind(): string;
	public getName(): string;
	public getVersion(): string;
	private matchField(field1: string, field2: string): boolean;
	public match(descriptor: Descriptor): boolean;
	private exactMatchField(field1: string, field2: string): boolean;
	public exactMatch(descriptor: Descriptor): boolean;
	public isComplete(): boolean;

public equals(value: any): boolean;
	public toString(): string;
	public static fromString(value: String): Descriptor;
}



```

```
pip-services:logger:console:logger1:1.0
pip-services:logger:elasticsearch:logger2:1.0
my-library:logger:mylog:logger3:1.0

```

```
*:logger:*:*:1.0

```

```
*:logger:console:*:1.0

```

```
*:logger:*:logger2:1.0

```

```
my_library:*:*:*:*

```

```
class SimpleController implements IReferenceable, IUnreferenceable {
  ...
      public setReferences(references) {
          this._worker = this._references.getOneRequired(
              new Descriptor("*", "worker", "worker1", "*", "1.0")
          );
      }
  ...
  }
  
let references = References.fromTuples(
    new Descriptor("sample", "worker", "worker1", "111", "1.0"), new Worker1("name1"),
    new Descriptor("sample", "worker", "worker2", "222", "1.0"), new Worker2("name1")
);

let controller = new SimpleController();
controller.setReferences(references);
controller.greeting("world");
controller.unsetReferences();
controller = null;
```

```
class DependencyResolver implements IReferenceable, IReconfigurable {
	private _dependencies: any = {};
	private _references: IReferences;
public configure(config: ConfigParams): void;
	public setReferences(references: IReferences): void;
	public put(name: string, locator: any): void;
	private locate(name: string): any;
	public getOptional<T>(name: string): T[];
	public getRequired<T>(name: string): T[];
	public getOneOptional<T>(name: string): T;
	public getOneRequired<T>(name: string): T;
	public find<T>(name: string, required: boolean): T[];
	public static fromTuples(...tuples: any[]): DependencyResolver;
}



```

```
class SimpleController implements IConfigurable, IReferenceable, IUnreferenceable {
  private _worker: any;
  
  private _depedencyResolver = DependencyResolver.fromTuples(
      "worker", new Descriptor("*","worker","*","*","1.0")
  );

  public configure(config) {
    this._depedencyResolver.configure(config);
  }

  public setReferences(references) {
    this._depedencyResolver.setReferences(references);
    this._worker = this._depedencyResolver.getOneRequired("worker");
  }

  public unsetReferences() {
    this._depedencyResolver = new DependencyResolver();
  }
...
}

let references = References.fromTuples(
  new Descriptor("sample", "worker", "worker1", "111", "1.0"), new Worker1("name1"),
  new Descriptor("sample", "worker", "worker2", "222", "1.0"), new Worker2("name1")
);

let config = ConfigParams.fromTuples(
  "dependencies.worker", "*:worker:worker1:111:1.0"
);

let controller = new SimpleController();
controller.configure(config);
controller.setReferences(references);
controller.greeting("world");
controller.unsetReferences();
controller = null;
```

```
- descriptor: "sample-references:worker:worker1:*:1.0"
  default_name: "Worker1"

- descriptor: "sample-references:worker:worker2:*:1.0"
  default_name: "Worker2"

- descriptor: "sample-references:controller:default:default:1.0"
  default_name: "Sample"
  dependencies:
    workers: "sample-references:worker:worker2:*:1.0"


```

```
class Referencer {
	public static setReferencesForOne(references: IReferences, component: any): void;
	public static setReferences(references: IReferences, components: any[]): void;
	public static unsetReferencesForOne(component: any): void;
	public static unsetReferences(components: any[]): void;
}


```
