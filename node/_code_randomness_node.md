```
import { RandomArray } from "pip-services4-data-node"

export function main() {
    let value1 = RandomArray.pick([ 1, 2, 3, 4 ]); // Possible result: 3
}
```

```
import { RandomBoolean } from "pip-services4-data-node"

export function main() {
    let value1 = RandomBoolean.nextBoolean();   // Possible result: True
    let value2 = RandomBoolean.chance(1, 3);    // Possible result: False
}
```

```
import { RandomDateTime } from "pip-services4-data-node"

export function main() {
    // Possible result: 2015-01-05 00:00:00
    let value1 = RandomDateTime.nextDate(new Date(2010, 1, 1));

    // Possible result: 2012-01-03
    let value2 = RandomDateTime.nextDate(new Date(2010, 1, 1), new Date(2015, 1, 1));

    // Possible result: 2020-03-11 11:20:32
    let value3 = RandomDateTime.nextDate(new Date(2017, 1, 1));

    // Possible result: 2010-01-02 00:00:01
    let value4 = RandomDateTime.updateDateTime(new Date(2010, 1, 2), 50); 
}
```

```
import { RandomDouble } from "pip-services4-data-node"

export function main() {
    // Random value between 5 and 10
    let value1 = RandomDouble.nextDouble(5, 10);      // Possible result: 7.3252274575446155

    // Random value lower than 10
    let value2 = RandomDouble.nextDouble(10);      // Possible result: 8.104104435251225

    // Random value 
    let value3 = RandomDouble.updateDouble(10, 5);      // Possible result: 8.051623143638789
}
```

```
import { RandomFloat } from "pip-services4-data-node"

export function main() {
    // Random value between 5 and 10
    let value1 = RandomFloat.nextFloat(5, 10);     // Possible result: 8.109282778264653

    // Random value lower than 10
    let value2 = RandomFloat.nextFloat(10);        // Possible result: 5.281760648272185

    // Random value
    let value3 = RandomFloat.updateFloat(10, 3);   // Possible result: 7.731874405844179
}
```

```
import { RandomInteger } from "pip-services4-data-node"

export function main() {
    // Random value between 5 and 10
    let value1 = RandomInteger.nextInteger(5, 10);     // Possible result: 5

    // Random value lower than 10
    let value2 = RandomInteger.nextInteger(10);        // Possible result: 4

    // Random value
    let value3 = RandomInteger.updateInteger(10, 3);   // Possible result: 12
}
```

```
import { RandomString } from "pip-services4-data-node"

export function main() {
    let value1 = RandomString.distort("hello John"); // Possible result: Hello john
    let value2 = RandomString.nextAlphaChar(); // Possible result: C
    let value3 = RandomString.nextString(5, 10); // Possible result .h*_N9}
    let value4 = RandomString.pick([ "A", "B", "C", "D", "E" ]); // Possible result: c
}
```

```
import { RandomText } from "pip-services4-data-node"

export function main() {
    let value1 = RandomText.adjective();    // Possible result: Small
    let value2 = RandomText.color();        // Possible result: White
    let value3 = RandomText.email();        // Possible result: NickStay @Hotel.com
    let value4 = RandomText.fullName();     // Possible result; Dr.Pamela Smith
    let value5 = RandomText.noun();         // Possible result: Car
    let value6 = RandomText.phone();        // Possible result: (147) 371 - 7084
    let value7 = RandomText.phrase(3);      // Possible result: Large
    let value8 = RandomText.word();         // Possible result: Hurry
    let value9 = RandomText.verb();         // Possible result: Lay
    let value10 = RandomText.text(5, 20);   // Possible result: Small carmack large
}
```
