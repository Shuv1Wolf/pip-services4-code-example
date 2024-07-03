```
import { 
    ConfigParams, Descriptor, IConfigurable, IOpenable, 
    IReferenceable, IReferences, IUnreferenceable, Context 
} from "pip-services4-components-node"

export class ComponentB implements IReferenceable, IConfigurable, IOpenable, IUnreferenceable {
    private _param1: string = 'ABC2'
    private _param2: number = 456
    private _opened = false
    private _status: string

    /**
     * Creates a new instance of the component.
     */
    public constructor() {
        this._status = "Created";
        console.log("ComponentB has been created.");
    }

    public configure(config: ConfigParams): void {
        this._param1 = config.getAsStringWithDefault("param1", this._param1);
        this._param2 = config.getAsIntegerWithDefault("param2", this._param2);
        console.log("ComponentB has been configured.");
    }

    public setReferences(references: IReferences): void {
        throw new Error("Method not implemented.");
    }

    public isOpen(): boolean {
        throw new Error("Method not implemented.");
    }
  
    public open(ctx: Context): Promise<void> {
        throw new Error("Method not implemented.");
    }

    public close(ctx: Context): Promise<void> {
        throw new Error("Method not implemented.");
    }

    /**
     * Unsets (clears) previously set references to dependent components.
     */
    public unsetReferences(): void {
        throw new Error("Method not implemented.");
    }
}

export class ComponentA1 implements IReferenceable, IConfigurable, IOpenable, IUnreferenceable {
    private _param1: string = 'ABC';
    private _param2: number = 123;
    private _another_component: ComponentB;
    private _opened: boolean = false;
    private dummy_variable: string;

    /**
    * Creates a new instance of the component.
    */
    public constructor() {
        console.log("ComponentA1 has been created.");
    }

    public setReferences(references: IReferences): void {
        this._another_component = references.getOneRequired(
            new Descriptor("myservice", "component-b", "*", "*", "1.0")
        )
        console.log("ComponentA1's references have been defined.");
    }

    public configure(config: ConfigParams): void {
        this._param1 = config.getAsStringWithDefault("param1", 'ABC');
        this._param2 = config.getAsIntegerWithDefault("param2", 123);
        console.log("ComponentA1 has been configured.");
    }

    public isOpen(): boolean {
        return this._opened;
    }

    public async open(ctx: Context): Promise<void> {
        this._opened = true;
        console.log("ComponentA1 has been opened.");
    }

    public async close(ctx: Context): Promise<void> {
        this._opened = false;
        console.log("ComponentA1 has been closed.");
    }

    public async myTask(correlationId: string) {
        console.log("Doing my business task");
        this.dummy_variable = "dummy value";
    }

    /**
     * Unsets (clears) previously set references to dependent components.
     */
    public unsetReferences(): void {
        this._another_component = null;
        console.log("References cleared");
    }
}

export class ComponentA2 implements IReferenceable, IConfigurable, IOpenable, IUnreferenceable {
    private _param1 = 'ABC';
    private _param2 = 123;
    private _another_component: ComponentB;
    private _opened = false;
    private _status = "";
    private dummy_variable: string;

    /**
     * Creates a new instance of the component.
     */
    public constructor() {
        this._status = "Created";
        console.log("ComponentA2 has been created.");
    }

    public setReferences(references: IReferences): void {
        this._another_component = references.getOneRequired(
            new Descriptor("myservice", "component-b", "*", "*", "1.0")
        )
        this._status = "Configured";
        console.log("ComponentA2's references have been defined.");
    }

    public configure(config: ConfigParams): void {
        this._param1 = config.getAsStringWithDefault("param1", 'ABC');
        this._param2 = config.getAsIntegerWithDefault("param2", 123);
        this._status = "Configured";
        console.log("ComponentA2 has been configured.");
    }

    public isOpen(): boolean {
        return this._opened;
    }

    public async open(ctx: Context): Promise<void> {
        this._opened = true;
        this._status = "Open";
        console.log("ComponentA2 has been opened.");
    }

    public async close(ctx: Context): Promise<void> {
        this._opened = false
        this._status = "Closed"
        console.log("ComponentA2 has been closed.")
    }

    public async myTask(correlationId): Promise<void> {
        console.log("Doing my business task");
        this.dummy_variable = "dummy value";
    }
  
    /**
     * Unsets (clears) previously set references to dependent components.
     */
    public unsetReferences(): void {
        this._another_component = null;
        this._status = "Un-referenced";
        console.log("References cleared");
    }
}
```

```
import { Descriptor, Factory } from "pip-services4-components-node";
import { ProcessContainer } from "pip-services4-container-node";


/**
 * Creating a process container
 */
export class MyProcess extends ProcessContainer {
    public constructor() {
        super('myservice', 'My service running as a process')
        this._configPath = './configV4.yaml'

        let MyFactory1 = new Factory();

        MyFactory1.registerAsType(new Descriptor("myservice", "component-a1", "default", "*", "1.0"), ComponentA1);
        MyFactory1.registerAsType(new Descriptor("myservice", "component-a2", "default", "*", "1.0"), ComponentA2);
        MyFactory1.registerAsType(new Descriptor("myservice", "component-b", "default", "*", "1.0"), ComponentB);

        this._factories.add(MyFactory1)
    }
}

/**
 * Running the container
 */
function main() {
    let runner = new MyProcess();
    console.log("run");
    try {
        runner.run(process.argv);
    }
    catch(ex){
        console.log(ex)
    }
}
```
