```
import (
    grpccontrl "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/controllers"
)
```

```
import (
    grpcclnt "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/clients"
)
```

```
type MyData struct {
	Id      string `json:"id"`
	Key     string `json:"key"`
	Content string `json:"content"`
}

func NewMyData(id string, key string, content string) *MyData {
	return &MyData{
		Id:      id,
		Key:     key,
		Content: content,
	}
}
```

```

```
