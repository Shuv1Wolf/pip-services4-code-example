```
import { Descriptor } from "pip-services4-components-node";
```

```
import { Descriptor } from "pip-services4-components-node";

export function main() {
  // Locate all connectors(match by type)
  let locator = Descriptor.fromString("*:connector:*:*:*");

  // Locate all connectors for a specific microservice(match by group and type)
  locator = Descriptor.fromString("mygroup:connector:*:*:*"); 

  // Locate a specific component(match by name)
  locator = Descriptor.fromString("*:*:*:my_name:*");
}
```

```
import { Descriptor } from "pip-services4-components-node";

export function main() {
  let locator = new Descriptor("mygroup", "connector", "aws", "default", "1.0");

  console.log(locator.getGroup());   // returns "my_group"
  console.log(locator.getKind());    // returns "aws"
  console.log(locator.getName());    // returns "default"
  console.log(locator.getVersion()); // returns "1.0"
}
```

```
import { Descriptor } from "pip-services4-components-node";

export function main() {
  let locator1 = new Descriptor("mygroup", "connector", "aws", "default", "1.0");
  console.log(locator1.toString());
}
```

```
import { Descriptor } from "pip-services4-components-node";

export function main() {
  let locator1 = new Descriptor("mygroup", "connector", "aws", "default", "1.0");
  let locator2 = Descriptor.fromString("mygroup:connector:*:*:1.0");

  locator1.isComplete();   // returns True
  locator2.isComplete();   // returns False
}
```

```
import { Descriptor } from "pip-services4-components-node";

export function main() {
  let locator1 = new Descriptor("mygroup", "connector", "aws", "default", "1.0");
  let locator2 = Descriptor.fromString("mygroup:connector:*:*:1.0");
  let locator3 = Descriptor.fromString("mygroup:connector:aws:default:1.0");

  locator1.match(locator2);       // returns True
  locator1.exactMatch(locator2);  // returns False
  locator1.equals(locator3);      // returns True
}
```

```
import { Descriptor, Factory } from "pip-services4-components-node";

export function main() {
  let MyFactory1 = new Factory();
  let classADescriptor = new Descriptor("mygroup", "class", "classA", "classA", "1.0");

  MyFactory1.registerAsType(classADescriptor, ClassA);

  MyFactory1.create(classADescriptor);
}

class ClassA{
  public constructor() {
      console.log("ClassA created");
  }
}
```
