
```python
from pip_services4_datadog.count import DataDogCounters
```

```python
class MyComponentA:
    
    _console_log = True
    
    def __init__(self, counters: DataDogCounters):
        self.counters = counters

        if self._console_log:
            print("MyComponentA has been created.")

    def mymethod(self):
        self.counters.increment("mycomponent.mymethod.calls", 1)
        timing = self.counters.begin_timing("mycomponent.mymethod.exec_time")
        try:
            if self._console_log:
                print("Hola amigo")
                print("Bonjour mon ami")
        finally:
            timing.end_timing()
        self.counters.dump()

```

```python

from pip_services4_components.config import ConfigParams

counters = DataDogCounters()
counters.configure(ConfigParams.from_tuples(
   "credential.access_key", "e1be2e70036b8f05f2777f5f038fc222"
))

counters.open(ctx)

```

```python
mycomponent = MyComponentA(counters)
for i in range(5):
    mycomponent.mymethod()
```

```python
import time

from typing import Optional

from pip_services4_components.config import IConfigurable, ConfigParams
from pip_services4_components.run import IOpenable
from pip_services4_components.context import Context
from pip_services4_datadog.count import DataDogCounters


class MyComponentA(IConfigurable, IOpenable):
    _console_log = True

    def __init__(self):
        self._counters = DataDogCounters()

        if self._console_log:
            print("MyComponentA has been created.")

    def configure(self, config: ConfigParams):
        self._counters.configure(config)

    def get_counters(self) -> DataDogCounters:
        return self._counters

    def is_open(self) -> bool:
        return self._counters.is_open()

    def open(self, ctx: Context):
        self._counters.open(ctx)

    def close(self, ctx: Context):
        self._counters.close(ctx)

    def my_method(self):
        self._counters.increment("mycomponent.mymethod.calls", 1)
        timing = self._counters.begin_timing("mycomponent.mymethod.exec_time")
        try:
            if self._console_log:
                print("Hola amigo")
                print("Bonjour mon ami")
        finally:
            timing.end_timing()
        self._counters.dump()


```

```python
from pip_services4_datadog.log import DataDogLogger
```

```python
from pip_services4_datadog.log import DataDogLogger

class MyComponentA:
    _Datadog_log = True
    
    def __init__(self, logger: DataDogLogger):
        self._logger = logger

        if self._Datadog_log:
            logger.info(ctx , "MyComponentA has been created.")

    def mymethod(self):

        try:
            if self._Datadog_log:
                print("Hola amigo")
                print("Bonjour mon ami")
                logger.info(ctx , "Greetings created.")
        finally:
                logger.info(ctx , "Finally reached.")

```

```python
from pip_services4_components.config import ConfigParams

logger = DataDogLogger()
logger.configure(ConfigParams.from_tuples(
   "credential.access_key", "e1be2e70036b8f05f2777f5f038fc222"
))
```

```python
mycomponent = MyComponentA(logger)
for i in range(5):
    mycomponent.mymethod()
```

```python
from pip_services4_datadog.log import DataDogLogger
from pip_services4_components.run import IOpenable
from pip_services4_components.context import Context
from pip_services4_components.config import IConfigurable, ConfigParams

from typing import Optional

class MyComponentA(IConfigurable, IOpenable):

    _Datadog_log = True
    
    def __init__(self):
        self._logger = DataDogLogger()

        if self._Datadog_log:
            self._logger.info(null , "MyComponentA has been created.")

    def configure(self, config: ConfigParams):
        self._logger.configure(config)
        
    def open(self, ctx: Context):
        self._logger.open(ctx)
       
    def close(self, ctx: Context):
        self._logger.close(ctx)
        
    def mymethod(self):

        try:
            if self._Datadog_log:
                print("Hola amigo")
                print("Bonjour mon ami")
                self._logger.info(ctx, "Greetings created.")
        finally:
                self._logger.info(ctx, "Finally reached.")

```