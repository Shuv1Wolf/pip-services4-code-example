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
import { IReferenceable, IReferences, Context } from "pip-services4-components-node";
import { CompositeLogger } from "pip-services4-observability-node";

class MyComponent implements IReferenceable {
    private _logger: CompositeLogger = new CompositeLogger();

    public setReferences(refs: IReferences) {
        this._logger.setReferences(refs);
    }

    public doSomething(ctx: Context) {
        this._logger.debug(ctx, "Did something...");
        ...
    }
}
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
import { IReferenceable, IReferences, Context } from "pip-services4-components-node";
import { CompositeCounters } from "pip-services4-observability-node";
import { MessageEnvelope } from "pip-services4-messaging-node"

class MyComponent implements IReferenceable {
    private _counters: CompositeCounters = new CompositeCounters();
  
    public setReferences(refs: IReferences) {
      this._counters.setReferences(refs);
    }
  
    public onMessage(message: MessageEnvelope) {
      let timing = this._counters.beginTiming("mycomponent:msg_time");
      try {
        this._counters.increment("mycomponent:msg_count", 1);
        ...
      } catch (ex) {
        this._counters.increment("mycomponent:msg_errors", 1);
      } finally {
        timing.endTiming();
      }
    }
  }
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
import { IReferenceable, IReferences, Context } from "pip-services4-components-node";
import { CompositeTracer } from "pip-services4-observability-node";

class MyComponent implements IReferenceable {
    private _tracer: CompositeTracer = new CompositeTracer();
  
    public setReferences(refs: IReferences) {
      this._tracer.setReferences(refs);
    }
  
    public doSomething(ctx: Context) {
      let timing = this._tracer.beginTrace(ctx, "mycomponent", "do_something");
      try {
        ...
        timing.endTrace();
      } catch (ex) {
        timing.endFailure(ex);
      }
    }
  }
```
