```
import { Factory } from "pip-services4-components-node";
```

```

export class MyComponent1 {
    private _status: string;
    /**
     * Creates a new instance of the component.
     */
    public constructor() {
        this._status = "Created";
        console.log("MyComponent1 has been created.");
    }

    public async myTask() {
        console.log("task executed")
    }
}

```

```
let factory = new Factory();
factory.registerAsType(new Descriptor("mygroup", "mycomponent1", "default", "*", "1.0"), MyComponent1);
let component1 = factory.create(new Descriptor("mygroup", "mycomponent1", "default", "name1", "1.0"));
```

```
component1.myTask();
```

```
import { Factory } from "pip-services4-components-node";
import { DefaultLogicFactory } from "pip-services4-logic-node";
```

```
let lockFactory = new DefaultLogicFactory();
```

```
let lock = lockFactory.create(new Descriptor("pip-services", "lock", "memory", "*", "1.0"));

```

```
typeof(lock)

```

```
import { CompositeFactory } from "pip-services4-components-node";
```

```
let compositeFactory = new CompositeFactory();
```

```
import { Descriptor, Factory  } from "pip-services4-components-node";

let factory1 = new Factory()
factory1.registerAsType(new Descriptor("mygroup", "mycomponent1", "default", "*", "1.0"), MyComponent1)
compositeFactory.add(factory1)
```

```
import { DefaultLogicFactory } from "pip-services4-logic-node";

compositeFactory.add(new DefaultLogicFactory());
```

```
let component1Locator = new Descriptor("*", "mycomponent1", "*", "*", "1.0");
component1 = compositeFactory.create(component1Locator);
```

```
let loggerLocator = new Descriptor("*", "logger", "console", "*", "1.0");

let result1 = compositeFactory.create(loggerLocator);
result1

```
