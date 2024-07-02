```
export interface IConfigurable {
  configure(config: ConfigParams): void;
}

export interface IReferenceable {
  setReferences(references: IReferences): void;
}

export interface IOpenable extends IClosable {

  isOpen(): boolean;

  open(ctx: Context): Promise<void>;
}

export interface IClosable {
  close(ctx: Context): Promise<void>;
}

export interface IExecutable {
  execute(ctx: Context, args: Parameters): Promise<any>;
}
```

```
import { ConfigParams, FixedRateTimer, Parameters, Context } from "pip-services4-components-node";
import { IReconfigurable, IReferenceable, IExecutable, IOpenable, IReferences } from "pip-services4-components-node";
import { CompositeLogger, LogLevel } from "pip-services4-observability-node";

class CounterController implements IReferenceable, IReconfigurable, IOpenable, IExecutable {
  

    private logger = new CompositeLogger();
    private timer = new FixedRateTimer();
    private parameters = new Parameters();
    private counter = 0;
  
    public constructor(){
        this.logger.setLevel(LogLevel.Debug);
    }

    public configure(config: ConfigParams){
        this.parameters = Parameters.fromConfig(config);
    }
    

    public setReferences(references: IReferences){
        this.logger.setReferences(references);
    }
    

    public isOpen(): boolean{
        return this.timer.isStarted();
    }

    public async open(ctx: Context): Promise<void> {
        if (this.isOpen()) {
            return;
        }

        this.timer.setCallback(() => this.execute(ctx, this.parameters))
        this.timer.setInterval(1000)
        this.timer.setDelay(1000)
        this.timer.start()
        this.logger.trace(ctx, "Counter controller opened")

    }
    
    public async close(ctx: Context): Promise<void> {
        this.timer.stop()
        this.logger.trace(ctx, "Counter controller closed")
    }
    

    public execute(ctx: Context, args: Parameters): Promise<any> {
        this.counter += 1;

        this.logger.info(
            ctx,
            this.counter + " - " + this.parameters.getAsStringWithDefault('message', 'Hello World!')
        )

        let result = args.getAsObject("message");
        return result;
    }
}
```

```
await Opener.open(ctx, references.getAll());
...
await Closer.close(ctx, references.getAll());

```
