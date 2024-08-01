
```python
from pip_services4_elasticsearch.log import ElasticSearchLogger
```

```python
class MyComponentA:

    _console_log = True
    
    def __init__(self, logger: ElasticSearchLogger):
        self._logger = logger

        self._logger.info(ctx, "MyComponentA has been created.")

    def mymethod(self):

        try:
            if self._console_log:
                print("Hola amigo")
                print("Bonjour mon ami")
            self._logger.info(ctx, "Greetings created.")
        finally:
                self._logger.info(ctx, "Finally reached.")
```

```python

logger = ElasticSearchLogger()
logger.configure(ConfigParams.from_tuples(
    "connection.protocol", "http",
    "connection.host", "localhost",
    "connection.port", 9200
))
logger.open(ctx)
```

```python
mycomponent = MyComponentA(logger)
for i in range(10):
    mycomponent.mymethod()
```

```python
from pip_services4_elasticsearch.log import ElasticSearchLogger
from pip_services4_components.run import IOpenable
from pip_services4_components.context import Context
from pip_services4_components.config import IConfigurable, ConfigParams
from typing import Optional
import time

class MyComponentA(IConfigurable, IOpenable):

    _console_log = True
    
    def __init__(self):
        self._logger = ElasticSearchLogger()

        self._logger.info(ctx, "MyComponentA has been created.")

    def configure(self, config: ConfigParams):
        self._logger.configure(config)
        
    def open(self, ctx: Context):
        self._logger.open(correlation_id)
       
    def close(self, ctx: Context):
        self._logger.close(correlation_id)
        
    def mymethod(self):

        try:
            if self._console_log:
                print("Hola amigo")
                print("Bonjour mon ami")
                self._logger.info(ctx, "Greetings created.")
        finally:
                self._logger.info(ctx, "Finally reached.")
```
