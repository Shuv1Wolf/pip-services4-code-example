````
type IConfigurable interface {
	Configure(ctx context.Context, config *ConfigParams)
}

type IReferenceable interface {
	SetReferences(ctx context.Context, references IReferences)
}

type IOpenable interface {
	IClosable

	IsOpen() bool

	Open(ctx context.Context) error
}


type IClosable interface {
	Close(ctx context.Context) error
}


type IExecutable interface {
	Execute(ctx context.Context, args *Parameters) (result interface{}, err error)
}
````

```
import (
	"context"

	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	clog "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/log"
)

type CounterController struct {
	logger     *clog.CompositeLogger
	timer      *cexec.FixedRateTimer
	parameters *cexec.Parameters
	counter    int
}

func NewCounterController() *CounterController {
	instance := &CounterController{
		logger:     clog.NewCompositeLogger(),
		timer:      cexec.NewFixedRateTimer(),
		parameters: cexec.NewEmptyParameters(),
		counter:    0,
	}

	instance.logger.SetLevel(clog.LevelDebug)

	return instance
}

func (c *CounterController) SetReferences(ctx context.Context, references crefer.IReferences) {
	c.logger.SetReferences(ctx, references)
}

func (c *CounterController) IsOpen() bool {
	return c.timer.IsStarted()
}

func (c *CounterController) Open(ctx context.Context) error {
	if c.IsOpen() {
		return nil
	}

	c.timer.SetCallback(func(ctx context.Context) { c.Execute(context.Background(), *c.parameters) })
	c.timer.SetInterval(1000)
	c.timer.SetDelay(1000)
	c.timer.Start(context.Background())
	c.logger.Trace(context.Background(), "Counter controller opened")

	return nil
}

func (c *CounterController) Close(ctx context.Context) error {
	c.timer.Stop(ctx)
	c.logger.Trace(context.Background(), "Counter controller closed")

	return nil
}

func (c *CounterController) Execute(ctx context.Context, args cexec.Parameters) (result interface{}, err error) {
	message, ok := args.GetAsObject("message")
	if !ok {
		return nil, nil
	}
	return message, nil
}
```

```
import (
      crun "github.com/pip-services4/pip-services4-go/pip-services4-components-go/run"
      crefer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

err := crun.Opener.Open(context.Background(), crefer.NewEmptyReferences().GetAll())
// ...
err = crun.Closer.Close(context.Background(), crefer.NewEmptyReferences().GetAll())
```
