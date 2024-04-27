### Logging

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
import (
	"context"

	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

type MyComponent struct {
	// ...
	logger *clog.CompositeLogger
}

func (c *MyComponent) SetReferences(ctx context.Context, references cref.IReferences) {
	c.logger.SetReferences(ctx, references)
}

func (c *MyComponent) DoSomething(ctx context.Context, correlationId string) {
	c.logger.Debug(ctx, correlationId, "Did something...")
	// ...
}

```

### Performance monitoring[ ](http://docs.pipservices.org/v3/tutorials/beginner_tutorials/building_blocks/observability/#performance-monitoring)

```
# Log counters
descriptor: "pip-services:counters:log:default:1.0"

# Prometheus counters
descriptor: "pip-services:counters:prometheus:default:1.0"

# Metrics service used by prometheus to collect metrics
descriptor: "pip-services:metrics-service:prometheus:default:1.0"

```

```
import (
	"context"

	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cmsg "github.com/pip-services4/pip-services4-go/pip-services4-messaging-go/queues"
	ccount "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/count"
)

type MyComponent struct {
	// ...
	counters *ccount.CompositeCounters
}

func (c *MyComponent) SetReferences(ctx context.Context, references cref.IReferences) {
	c.counters.SetReferences(ctx, references)
}

func (c *MyComponent) OnMessage(ctx context.Context, message *cmsg.MessageEnvelope) {
	timing := c.counters.BeginTiming(ctx, "mycomponent:msg_time")
	defer timing.EndTiming(ctx)

	c.counters.Increment(ctx, "mycomponent:msg_count", 1)
	// ...

	if err != nil {
		c.counters.Increment(ctx, "mycomponent:msg_errors", 1)
	}
}
```

### Traces

```
# Log tracer
descriptor: "pip-services:tracer:log:default:1.0"

# DataDog traces
descriptor: "pip-services:tracer:datadog:default:1.0"
connection:
  api_key: {{DATADOG_API_KEY}}
```

```
import (
	"context"

	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	ctrace "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/trace"
)

type MyComponent struct {
	// ...
	tracer *ctrace.CompositeTracer
}

func (c *MyComponent) SetReferences(ctx context.Context, references cref.IReferences) {
	c.tracer.SetReferences(ctx, references)
}

func (c *MyComponent) DoSomething(ctx context.Context) {
	timing := c.tracer.BeginTrace(ctx, "mycomponent", "do_something")

	// ...

	if err != nil {
		timing.EndFailure(err)
	} else {
		timing.EndTrace()
	}
}
```
