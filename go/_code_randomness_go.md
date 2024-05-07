```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	value1 := rand.Array.Pick([]int{1, 2, 3, 4}) // Possible result: 3
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	value1 := rand.Boolean.Next()       // Possible result: True
	value2 := rand.Boolean.Chance(1, 3) // Possible result: False
}
```

```
import (
	"time"

	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	maxDate := time.Date(2050, 1, 1, 0, 0, 0, 0, time.Local)

	// Possible result: 2015-01-05 00:00:00
	value1 := rand.DateTime.NextDate(time.Date(2010, 1, 1, 0, 0, 0, 0, time.Local), maxDate)

	// Possible result: 2012-01-03
	value2 := rand.DateTime.NextDate(time.Date(2010, 1, 1, 0, 0, 0, 0, time.Local), time.Date(2015, 1, 1, 0, 0, 0, 0, time.Local))

	// Possible result: 2020-03-11 11:20:32
	value3 := rand.DateTime.NextDateTime(time.Date(2017, 1, 1, 0, 0, 0, 0, time.Local), maxDate)

	// Possible result: 2010-01-02 00:00:01
	value4 := rand.DateTime.UpdateDateTime(time.Date(2010, 1, 2, 0, 0, 0, 0, time.Local), 500)
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	value1 := rand.Double.Next(5, 10) // Possible result: 7.3252274575446155

	// Random value lower than 10
	value2 := rand.Double.Next(10, 11) // Possible result: 10.104104435251225

	// Random value
	value3 := rand.Double.Update(10, 5) // Possible result: 8.051623143638789
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	// Random value between 5 and 10
	value1 := rand.Float.Next(5, 10) // Possible result: 8.109282778264653

	// Random value lower than 10
	value2 := rand.Float.Next(10, 15) // Possible result: 14.281760648272185

	// Random value
	value3 := rand.Float.Update(10, 3) // Possible result: 7.73187440584417
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	// Random value between 5 and 10
	value1 := rand.Integer.Next(5, 10) // Possible result: 7

	// Random value lower than 10
	value2 := rand.Integer.Next(10, 15) // Possible result: 11

	// Random value
	value3 := rand.Integer.Update(10, 3) // Possible result: 10
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	value1 := rand.String.Distort("hello John")                   // Possible result: Hello john
	value2 := rand.String.NextAlphaChar()                         // Possible result: C
	value3 := rand.String.Next(5, 10)                             // Possible result .h*_N9}
	value4 := rand.String.Pick([]string{"A", "B", "C", "D", "E"}) // Possible result: c
	value5 := rand.String.PickChar("Jonathan")                    // Possible result: n
}
```

```
import (
	rand "github.com/pip-services4/pip-services4-go/pip-services4-data-go/random"
)

func main() {
	value1 := rand.Text.Adjective()  // Possible result: Small
	value2 := rand.Text.Color()      // Possible result: White
	value3 := rand.Text.Email()      // Possible result: NickStay@Hotel.com
	value4 := rand.Text.FullName()   // Possible result; Dr. Pamela Smith
	value5 := rand.Text.Word()       // Possible result: Car
	value6 := rand.Text.Phone()      // Possible result: (147) 371-7084
	value7 := rand.Text.Phrase(3, 7) // Possible result: Large
	value8 := rand.Text.Words(3, 5)  // Possible result: HurryJohns
	value9 := rand.Text.Verb()       // Possible result: Lay
	value10 := rand.Text.Text(5, 20) // Possible result: Small carmack large
}
```

```

```
