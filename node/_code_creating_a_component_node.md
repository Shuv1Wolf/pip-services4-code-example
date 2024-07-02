```
class MyComponentA {
    private _param1: string = "ABC";
    private _param2: number = 123;
    private _open: boolean = false;
    private _status: string;

    // Creates a new instance of the component.
    public constructor(){
        this._status = "Created";
        console.log("MyComponentA has been created.");
    }
}
```

```
import { IConfigurable, ConfigParams } from "pip-services4-components-node";

class MyComponentA implements IConfigurable {
  private _param1: string = "ABC";
  private _param2: number = 123;
  private _open: boolean = false;
  private _status: string;

  // Creates a new instance of the component.
  public constructor(){
      this._status = "Created";
      console.log("MyComponentA has been created.");
  }

  public configure(config: ConfigParams): void {
      this._param1 = config.getAsStringWithDefault("param1", "ABC");
      this._param2 = config.getAsIntegerWithDefault("param2", 123);
      this._status = "Configured";

      console.log("MyComponentA has been configured.");
  }
}
```

```
import { 
  IConfigurable, ConfigParams, IReferences, 
  IReferenceable, Descriptor 
} from "pip-services4-components-node";

class MyComponentB {
  private _param1: string = "ABC2";
  private _param2: number = 456;
  private _open: boolean = false;
  private _status: string;

  // Creates a new instance of the component.
  public constructor() {
      this._status = "Created";
      console.log("MyComponentB has been created.");
  }
}

class MyComponentA implements IConfigurable, IReferenceable {
  private _param1: string = "ABC";
  private _param2: number = 123;
  private _open: boolean = false;
  private _status: string;
  private _anotherComponent: MyComponentB;

  // Creates a new instance of the component.
  public constructor(){
      this._status = "Created";
      console.log("MyComponentA has been created.");
  }

  public setReferences(references: IReferences): void {
      this._anotherComponent = references.getOneRequired<MyComponentB>(
          new Descriptor("myservice", "mycomponent-b", "*", "*", "1.0")
      );

      this._status = "Configured";
      console.log("MyComponentA's references have been defined.");
  }

  public configure(config: ConfigParams): void {
      this._param1 = config.getAsStringWithDefault("param1", "ABC");
      this._param2 = config.getAsIntegerWithDefault("param2", 123);
      this._status = "Configured";

      console.log("MyComponentA has been configured.");
  }
}
```

```
import { 
  IConfigurable, ConfigParams, IReferences, 
  IReferenceable, Descriptor, IOpenable, Context, IContext
} from "pip-services4-components-node";

class MyComponentB {
  // ...
}


class MyComponentA implements IConfigurable, IReferenceable, IOpenable {
  private _param1: string = "ABC";
  private _param2: number = 123;
  private _open: boolean = false;
  private _status: string;
  private _anotherComponent: MyComponentB;

  // Creates a new instance of the component.
  public constructor(){
      this._status = "Created";
      console.log("MyComponentA has been created.");
  }

  public isOpen(): boolean {
      return this._open;
  }

  public async open(ctx: Context): Promise<void> {
    return new Promise<void>((resolve) => {
      this._open = true;
      this._status = "Open";
      console.log("MyComponentA has been opened.");
      resolve();
  });
  }

  public setReferences(references: IReferences): void {
      this._anotherComponent = references.getOneRequired<MyComponentB>(
          new Descriptor("myservice", "mycomponent-b", "*", "*", "1.0")
      );

      this._status = "Configured";
      console.log("MyComponentA's references have been defined.");
  }

  public configure(config: ConfigParams): void {
      this._param1 = config.getAsStringWithDefault("param1", "ABC");
      this._param2 = config.getAsIntegerWithDefault("param2", 123);
      this._status = "Configured";

      console.log("MyComponentA has been configured.");
  }
}
```

```
import { 
  IConfigurable, IReferenceable, IOpenable
} from "pip-services4-components-node";

class MyComponentB {
  // ...
}
   
  
class MyComponentA implements IConfigurable, IReferenceable, IOpenable {
  private _param1: string = "ABC";
  private _param2: number = 123;
  private _open: boolean = false;
  private _status: string;
  private _anotherComponent: MyComponentB;
  
  public dummyVariable: string;

  // ...

  public myTask(correlationId: string): void {
      console.log("Doing my business task");
      this.dummyVariable = "dummy value";
  }
}
```

```
import { 
  IConfigurable, IReferenceable, IOpenable, Context
} from "pip-services4-components-node";

class MyComponentB {
  // ...
}
   
  
class MyComponentA implements IConfigurable, IReferenceable, IOpenable {
  private _param1: string = "ABC";
  private _param2: number = 123;
  private _open: boolean = false;
  private _status: string;
  private _anotherComponent: MyComponentB;
  
  public dummyVariable: string;

  // ...

  public close(ctx: Context): Promise<void> {
    return new Promise<void>((resolve) => {  
      this._open = false;
      this._status = "Closed";
      console.log("MyComponentA has been closed.");

      resolve();
    });
  }
}
```

```

```
