```
# Console logger
descriptor: "pip-services:logger:console:default:1.0"
level: info

# Elastic search logger
descriptor: "pip-services:logger:elasticsearch:default:1.0"
connection:
  host: {{ELASTICSEARCH_SERVICE_HOST}}
  port: {{ELASTICSEARCH_SERVICE_PORT}}
```

```
from pip_services4_components.config import ConfigParams, IConfigurable
from pip_services4_observability.log import CompositeLogger


class MyComponent(IReferenceable):
    _logger: CompositeLogger = CompositeLogger()

    def set_references(self, refs: IReferences):
        self._logger.set_references(refs)

    def do_something(self, correlation_id: str):
        self._logger.debug(correlation_id, "Did something...")
```

```
# Log counters
descriptor: "pip-services:counters:log:default:1.0"

# Prometheus counters
descriptor: "pip-services:counters:prometheus:default:1.0"

# Metrics service used by prometheus to collect metrics
descriptor: "pip-services:metrics-service:prometheus:default:1.0"

```

```
from pip_services4_components.refer import IReferenceable, IReferences
from pip_services4_observability.count import CompositeCounters
from pip_services4_messaging.queues import MessageEnvelope

class MyComponent(IReferenceable):
    _counters: CompositeCounters = CompositeCounters()

    def set_references(self, refs: IReferences):
        self._counters.set_references(refs)

    def on_message(self, message: MessageEnvelope):
        timing = self._counters.begin_timing("mycomponent:msg_time")
        try:
            self._counters.increment("mycomponent:msg_count", 1)
            ...
        except Exception as ex:
            self._counters.increment("mycomponent:msg_errors", 1)
        finally:
            timing.end_timing()
```

```
# Log tracer
descriptor: "pip-services:tracer:log:default:1.0"

# DataDog traces
descriptor: "pip-services:tracer:datadog:default:1.0"
connection:
  api_key: {{DATADOG_API_KEY}}


```

```
from pip_services4_components.refer import IReferenceable, IReferences
from pip_services4_observability.trace import CompositeTracer


class MyComponent(IReferenceable):
    _tracer: CompositeTracer = CompositeTracer()

    def set_references(self, refs: IReferences):
        self._tracer.set_references(refs)

    def do_something(self, correlation_id: str):
        timing = self._tracer.begin_trace(correlation_id, "mycomponent", "do_something")
        try:
            ...
            timing.end_trace()
        except Exception as ex:
            timing.end_failure(ex)
```
