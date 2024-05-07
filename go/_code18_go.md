```
import (
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

myCommandSetA := ccmd.NewCommandSet()
```

```
type MyCommandSet struct {
	*ccmd.CommandSet
}
```

```
import (
	"context"
	"fmt"

	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type MyCommandSet struct {
	*ccmd.CommandSet
}

func NewMyCommandSet() *MyCommandSet {
	c := &MyCommandSet{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddCommand(c.command1())
	return c
}

func (c *MyCommandSet) command1() ccmd.ICommand {
	return ccmd.NewCommand(
		"command1",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 1")
			return
		},
	)
}
```

```
import (
	"context"
	"fmt"

	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type MyCommandSet struct {
	ccmd.CommandSet
}

func NewMyCommandSet() *MyCommandSet {
	c := &MyCommandSet{
		CommandSet: *ccmd.NewCommandSet(),
	}

	c.AddCommands([]ccmd.ICommand{c.command2(), c.command3()})
	return c
}

func (c *MyCommandSet) command2() ccmd.ICommand {
	return ccmd.NewCommand(
		"command2",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 2")
			return
		},
	)
}

func (c *MyCommandSet) command3() ccmd.ICommand {
	return ccmd.NewCommand(
		"command3",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 3")
			return
		},
	)
}
```

```
type MyCommandSetB struct {
	*ccmd.CommandSet
}

func NewMyCommandSetB() *MyCommandSetB {
	c := &MyCommandSetB{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddCommand(c.command1B())
	return c
}

func (c *MyCommandSetB) command1B() ccmd.ICommand {
	return ccmd.NewCommand(
		"command1B",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 1B")
			return
		},
	)
}

type MyCommandSet struct {
	*ccmd.CommandSet
}

func NewMyCommandSet() *MyCommandSet {
	c := &MyCommandSet{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddCommandSet(NewMyCommandSetB().CommandSet)
	return c
}
```

```
mySet := NewMyCommandSet()

mySet.Execute(context.Background(), "", "command1", nil) // Returns command 1
```

```
result := mySet.FindCommand("command1")

fmt.Println(result.Name()) // Returns 'command1'
```

```
result2 := mySet.Commands()

for _, command := range result2 {
	fmt.Println(command.Name())
}

// Returns  
// command1
// command2
// command3
// command1B
```

```
type MyEventSet struct {
	*ccmd.CommandSet
}

func NewMyEventSet() *MyEventSet {
	c := &MyEventSet{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddEvent(c.event1())
	return c
}

func (c *MyEventSet) event1() ccmd.IEvent {
	return ccmd.NewEvent("event1")
}
```

```
type MyEventSet struct {
	*ccmd.CommandSet
}

func NewMyEventSet() *MyEventSet {
	c := &MyEventSet{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddEvents([]ccmd.IEvent{c.event2(), c.event3()})
	return c
}

func (c *MyEventSet) event2() ccmd.IEvent {
	return ccmd.NewEvent("event2")
}

func (c *MyEventSet) event3() ccmd.IEvent {
	return ccmd.NewEvent("event3")
}
```

```
type MyEventSet struct {
	*ccmd.CommandSet
}

func NewMyEventSet() *MyEventSet {
	c := &MyEventSet{
		CommandSet: ccmd.NewCommandSet(),
	}

	c.AddEvents([]ccmd.IEvent{c.event2(), c.event3()})
	c.AddListener(c.listener1())
	return c
}

func (c *MyEventSet) event2() ccmd.IEvent {
	return ccmd.NewEvent("event2")
}

func (c *MyEventSet) event3() ccmd.IEvent {
	return ccmd.NewEvent("event3")
}

func (c *MyEventSet) listener1() ccmd.IEventListener {
	return NewMyListener()
}

type MyListener struct{}

func (c *MyListener) OnEvent(correlationId string, event ccmd.IEvent, value *crun.Parameters) {
	fmt.Println("Fired event " + event.Name())
}

func NewMyListener() *MyListener {
	return &MyListener{}
}
```

```
result4 := myEvents.FindEvent("event1")

fmt.Println(result4.Name())  // Returns 'event1'
```

```

myEvents := NewMyEventSet()

result3 := myEvents.Events()

for _, event := range result3 {
	fmt.Println(event.Name())
}

// Returns:  
// event1
// event2
// event3
```

```
package main

import (
	"context"
	"fmt"

	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

func main() {

	mySet := NewMyCommandSet()

	mySet.Execute(context.Background(), "command1", nil)
	mySet.Execute(context.Background(), "command2", nil)
	mySet.Execute(context.Background(), "command3", nil)
	mySet.Execute(context.Background(), "command1B", nil)
}

type MyCommandSetB struct {
	ccmd.CommandSet
}

func NewMyCommandSetB() *MyCommandSetB {
	c := &MyCommandSetB{
		CommandSet: *ccmd.NewCommandSet(),
	}

	c.AddCommand(c.command1B())
	return c
}

func (c *MyCommandSetB) command1B() ccmd.ICommand {
	return ccmd.NewCommand(
		"command1B",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 1B")
			return
		},
	)
}

type MyCommandSet struct {
	ccmd.CommandSet
}

func NewMyCommandSet() *MyCommandSet {
	c := &MyCommandSet{
		CommandSet: *ccmd.NewCommandSet(),
	}

	c.AddCommand(c.command1())
	c.AddCommandSet(&NewMyCommandSetB().CommandSet)
	c.AddCommands([]ccmd.ICommand{c.command2(), c.command3()})
	return c
}

func (c *MyCommandSet) command1() ccmd.ICommand {
	return ccmd.NewCommand(
		"command1",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 1")
			return
		},
	)
}

func (c *MyCommandSet) command2() ccmd.ICommand {
	return ccmd.NewCommand(
		"command2",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 2")
			return
		},
	)
}

func (c *MyCommandSet) command3() ccmd.ICommand {
	return ccmd.NewCommand(
		"command3",
		nil,
		func(ctx context.Context, args *cexec.Parameters) (result interface{}, err error) {
			fmt.Println("command 3")
			return
		},
	)
}

```

```
package main

import (
	"context"
	"fmt"

	cexec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

// Step 1 - Create the command set with events
type MyEventSet struct {
	ccmd.CommandSet
}

func NewMyEventSet() *MyEventSet {
	c := &MyEventSet{
		CommandSet: *ccmd.NewCommandSet(),
	}

	c.AddEvent(c.event1())
	c.AddEvents([]ccmd.IEvent{c.event2(), c.event3()})
	c.AddListener(c.listener1())
	return c
}

func (c *MyEventSet) event1() ccmd.IEvent {
	return ccmd.NewEvent("event1")
}

func (c *MyEventSet) event2() ccmd.IEvent {
	return ccmd.NewEvent("event2")
}

func (c *MyEventSet) event3() ccmd.IEvent {
	return ccmd.NewEvent("event3")
}

func (c *MyEventSet) listener1() ccmd.IEventListener {
	return NewMyListener()
}

// Step 2 - Create a listener
type MyListener struct{}

func (c *MyListener) OnEvent(ctx context.Context, event ccmd.IEvent, value *cexec.Parameters) {
	fmt.Println("Fired event " + event.Name())
}

func NewMyListener() *MyListener {
	return &MyListener{}
}

func main() {
	// Step 3  - Create an instance of the command set
	myEvents := NewMyEventSet()

	// Step 4 - Obtain events
	event1 := myEvents.FindEvent("event1")
	events := myEvents.Events() // Returns a list with event1, event2 and event3

	// Step 5 - Select event1 (first element in the list)
	event2 := events[1] // Returns event1

	// Step 6 - Notify the listener of an event occurrence
	event1.Notify(context.Background(), nil)
	event2.Notify(context.Background(), nil)
	myEvents.Notify(context.Background(), "event3", nil)
}
```
