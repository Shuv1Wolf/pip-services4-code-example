```
import { ConfigParams, IConfigurable } from "pip-services4-components-node";

let config = ConfigParams.fromTuples(
    'param1', 'XYZ',
    'param2', 345
  );
  component.configure(config);
  
  /// Also, often components can have hard-coded presets. 
  /// The ConfigParams class has methods that allow to easily use them as defaults:
  
class MyComponent implements IConfigurable {
    private _param1: string = 'ABC';
    private _param2: number = 123;

    public configure(config: ConfigParams) {
        this._param1 = config.getAsStringWithDefault('param1', this._param1);
        this._param2 = config.getAsIntegerWithDefault('param2', this._param2);
    }
}
```

```
export MYCOMPONENT_ENABLED=true
export PARAM1=XYZ
node ./bin/main.js -p PARAM2=345

```

```
{{#if MYCOMPONENT_ENABLED}}
descriptor: myservice:mycomponent:default:default:1.0
param1: {{PARAM1}}
param2: {{PARAM2}}
{{/if}}
```
