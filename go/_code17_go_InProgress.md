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
    ccontext "github.com/pip-services4/pip-services4-go/pip-services4-components-go/context"
) ?????
```


```
