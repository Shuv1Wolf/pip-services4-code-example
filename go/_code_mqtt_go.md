```
import (
	mqttqueue "github.com/pip-services4/pip-services4-go/pip-services4-mqtt-go/queues"
)
```

```
queue := mqttqueue.NewMqttMessageQueue("mytopic")
```

```
queue := mqttqueue.NewMqttMessageQueue("")
```

```
import conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"

queue := mqttqueue.NewMqttMessageQueue("")
queue.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"topic", "mytopic", // set topic
	"connection.protocol", "mqtt",
	"connection.host", "localhost",
	"connection.port", 1883,
	"options.qos", "1", // sets the qos level to "At least once"
	"options.autosubscribe", true, // autosubscription on the topic
	"options.serialize_envelope", true, // converts object into mosquitto values
))
```

```
queue.Open(context.Background())
```

```
queue.Close(context.Background())
```

```
import msgqueues "github.com/pip-services4/pip-services4-go/pip-services4-messaging-go/queues"

queue.Send(context.Background(), msgqueues.NewMessageEnvelope("", "mymessage", []byte("ABC123")))
```

```
message, err := queue.Receive(context.Background(), 10000)
```

```
msgText := message.GetMessageAsString() // Returns 'ABC123'
```

```
receivedPeek, err := queue.Peek(context.Background())
```

```
msgText := receivedPeek.GetMessageAsString(); // Returns 'ABCD123'
```

```
receivedPeekBatch, err := queue.PeekBatch(context.Background(), 3)
```

```
msgText := receivedPeekBatch[0].GetMessageAsString() // Returns 'ABC123'

```

```
res, err := queue.ReadMessageCount() // Returns 1
```

```
queue.Configure(context.Background(), conf.NewConfigParamsFromTuples(
	"topic", "mytopic",                 // set topic
	"connection.protocol", "mqtt",
	"connection.host", "localhost",
	"connection.port", 1883,
	"options.autosubscribe", true,     	// autosubscription on the topic
	"options.serialize_envelope", true, // converts object into musquitto values 
))
```

```
import (
	"context"
	"fmt"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	msgqueues "github.com/pip-services4/pip-services4-go/pip-services4-messaging-go/queues"
	mqttqueue "github.com/pip-services4/pip-services4-go/pip-services4-mqtt-go/queues"
)

func main() {
	// Component creation and configuration
	queue := mqttqueue.NewMqttMessageQueue("")

	queue.Configure(context.Background(), conf.NewConfigParamsFromTuples(
		"topic", "mytopic", // set topic
		"connection.protocol", "mqtt",
		"connection.host", "localhost",
		"connection.port", 1883,
		"options.autosubscribe", true, // autosubscription on the topic
		"options.serialize_envelope", true, // converts object into musquitto values
	))

	// Connection
	queue.Open(context.Background())

	// Send a message
	queue.Send(context.Background(), msgqueues.NewMessageEnvelope("", "mymessage", []byte("ABC1234")))

	// Receive a message
	message, _ := queue.Receive(context.Background(), 10000)
	fmt.Println(message.GetMessageAsString())

	// Close the connection
	queue.Close(context.Background())
}
```
