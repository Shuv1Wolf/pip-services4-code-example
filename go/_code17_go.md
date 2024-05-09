```
import (
    cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)
```

```

import (
	"fmt"
)

type MyComponent1 struct {
	_status string
}

func NewMyComponent1() *MyComponent1 {
	c := &MyComponent1{}
	c._status = "Created"
	fmt.Println("MyComponent1 has been created.")
	return c
}

func MyTask() {
	fmt.Println("task executed")
}
```

```
factory := cbuild.NewFactory()
factory.RegisterType(cref.NewDescriptor("mygroup", "mycomponent1", "default", "*", "1.0"), NewMyComponent1)
component1, err = factory.Create(cref.NewDescriptor("mygroup", "mycomponent1", "default", "name1", "1.0"))

```

```
component1.MyTask()
```

```
import (
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	clogic "github.com/pip-services4/pip-services4-go/pip-services4-logic-go/build"
)
```

```
logicFactory := clogic.NewDefaultLogicFactory()
```

```
lock, err := logicFactory.Create(cref.NewDescriptor("pip-services", "lock", "memory", "*", "1.0"))
```

```
reflect.TypeOf(lock)
```

```
import (
    cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
)
```

```
compositeFactory := cbuild.NewCompositeFactory()
```

```
factory1 := cbuild.NewFactory()
factory1.RegisterType(cref.NewDescriptor("mygroup", "mycomponent1", "default", "*", "1.0"), NewMyComponent1)
compositeFactory.Add(factory1)
```

```
import (
      cobserv "github.com/pip-services4/pip-services4-go/pip-services4-observability-go/build"
)

compositeFactory.Add(cobserv.NewDefaultObservabilityFactory())
```

```
component1Locator := cref.NewDescriptor("*", "mycomponent1", "*", "*", "1.0")
component1, err := compositeFactory.Create(component1Locator)
```

```
loggerLocator := cref.NewDescriptor("*", "logger", "console", "*", "1.0")
result1, err := compositeFactory.Create(loggerLocator)
```
