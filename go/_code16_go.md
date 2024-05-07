```
import (
    refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)
```

```
import (
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	// Locate all connectors (match by type)
	locator, _ := refer.ParseDescriptorFromString("*:connector:*:*:*")

	// Locate all connectors for a specific microservice (match by group and type)
	locator, _ = refer.ParseDescriptorFromString("mygroup:connector:*:*:*")

	// Locate a specific component (match by name)
	locator, _ = refer.ParseDescriptorFromString("*:*:*:my_name:*")
}
```

```
import (
	"fmt"

	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	locator := refer.NewDescriptor("mygroup", "connector", "aws", "default", "1.0")

	fmt.Println(locator.Group())   // returns "my_group"
	fmt.Println(locator.Kind())    // returns "aws"
	fmt.Println(locator.Name())    // returns "default"
	fmt.Println(locator.Version()) // returns "1.0"
}
```

```
import (
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	locator1 := refer.NewDescriptor("mygroup", "connector", "aws", "default", "1.0")
	locator1.String()
}
```

```
import (
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	locator1 := refer.NewDescriptor("mygroup", "connector", "aws", "default", "1.0")
	locator2, _ := refer.ParseDescriptorFromString("mygroup:connector:*:*:1.0")

	locator1.IsComplete() // returns True
	locator2.IsComplete() // returns False
}
```

```
import (
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

func main() {
	locator1 := refer.NewDescriptor("mygroup", "connector", "aws", "default", "1.0")
	locator2, _ := refer.ParseDescriptorFromString("mygroup:connector:*:*:1.0")
	locator3, _ := refer.ParseDescriptorFromString("mygroup:connector:aws:default:1.0")

	locator1.Match(locator2)      // returns True
	locator1.ExactMatch(locator2) // returns False
	locator1.Equals(locator3)     // returns True
}
```

```
import (
	"fmt"

	build "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	refer "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type ComponentA struct{}

func NewComponentA() *ComponentA {
	defer fmt.Println("Created ComponentA")
	return &ComponentA{}
}

func main() {
	MyFactory1 := build.NewFactory()

	myComponentADescriptor := refer.NewDescriptor("mygroup", "class", "classA", "classA", "1.0")

	MyFactory1.RegisterType(myComponentADescriptor, NewComponentA)

	MyFactory1.Create(myComponentADescriptor)
}
```
