```
import { ObjectSchema } from "pip-services4-data-node";
import { TypeCode } from "pip-services4-commons-node";

class MyObjectSchema extends ObjectSchema {
    public constructor()
    {
        super();

        this.withRequiredProperty('prop1', TypeCode.Integer);
        this.withOptionalProperty('prop2', TypeCode.String);
        this.withOptionalProperty('prop3', new MySubObjectSchema());
    }
}
```

```
import { FilterParams } from "pip-services4-data-node";

let filter = FilterParams.fromTuples(
    'key1', 'ABC',
    'key2', 123
);

let values = await client.getMyObjects(filter);
```

```
import { SortParams, SortField } from "pip-services4-data-node";

let sorting = new SortParams(
    new SortField("key1", true),
    new SortField("key2", false)
  );
  
  let values = await client.getMyObjects(filter, sorting);
```

```
import { PagingParams } from "pip-services4-data-node";

let paging = new PagingParams(0, 100, true);

let page = await client.getMyObjects(filter, sorting, paging);
```

```
import { ProjectionParams } from "pip-services4-data-node";

let projection = new ProjectionParams(['field1', 'field2']);

let page = await client.getMyObjects(filter, sorting, paging, projection);
```
