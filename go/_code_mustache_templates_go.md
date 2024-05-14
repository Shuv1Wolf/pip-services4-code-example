```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-expressions-go@latest
```

```
import (
	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// variable
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}")
	variables := map[string]string{
		"NAME": "Alex",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// conditional construction
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}{{ #if EXCLAMATION }}!{{/if}}")
	variables := map[string]string{
		"NAME":        "Alex",
		"EXCLAMATION": "1",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// conditional construction
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}{{ #if EXCLAMATION }}!{{/if}}")
	variables := map[string]string{
		"NAME":        "Alex",
		"EXCLAMATION": "",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// "if else" construction
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}{{ #if EXCLAMATION }}!{{/if}}{{{^EXCLAMATION}}}.{{{/EXCLAMATION}}}")
	variables := map[string]string{
		"NAME":        "Alex",
		"EXCLAMATION": "",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// "if else" construction
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}{{ #if EXCLAMATION }}!{{/if}}{{{^EXCLAMATION}}}.{{{/EXCLAMATION}}}")
	variables := map[string]string{
		"NAME":        "Alex",
		"EXCLAMATION": "1",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	// equivalent constructions
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello, {{{NAME}}}{{{#unless EXCLAMATION}}}.{{{/unless}}}")
	variables := map[string]string{
		"NAME":        "Alex",
		"EXCLAMATION": "1",
	}

	result, _ := template.EvaluateWithVariables(variables)

	fmt.Println(result)
}
```

```
import (
	"fmt"

	mustache "github.com/pip-services4/pip-services4-go/pip-services4-expressions-go/mustache"
)

func main() {
	variable := map[string]string{
		"NAME":        "Joe ",
		"SURNAME":     "Smith",
		"ESCLAMATION": "",
	}
	template := mustache.NewMustacheTemplate()
	template.SetTemplate("Hello Mr, {{{NAME}}} {{{SURNAME}}}")
	template.SetDefaultVariables(variable)
	result, _ := template.Evaluate()

	fmt.Println(result)
}
```

```
template.SetDefaultVariables(
	map[string]string{
		"NAME":    "Joe ",
		"SURNAME": "Smith",
	},
)
```

```
template.Clear()
```
