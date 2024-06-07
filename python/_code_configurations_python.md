```
from pip_services4_components.config import ConfigParams, IConfigurable

config = ConfigParams.from_tuples(
  'param1', 'XYZ',
  'param2', 345
)

component.configure(config)

# Also, often components can have hard-coded presets. 
# The ConfigParams class has methods that allow to easily use them as defaults:

class MyComponent(IConfigurable):
    _param1: str = 'ABC'
    _param2: int = 123

    def configure(self, config: ConfigParams):
        self._param1 = config.get_as_string_with_default('param1', self._param1)
        self._param2 = config.get_as_integer_with_default('param2', self._param2)

```

```
export MYCOMPONENT_ENABLED=true
export PARAM1=XYZ
python ./bin/main.py -p PARAM2=345

```

```
{{#if MYCOMPONENT_ENABLED}}
descriptor: myservice:mycomponent:default:default:1.0
param1: {{PARAM1}}
param2: {{PARAM2}}
{{/if}}
```
