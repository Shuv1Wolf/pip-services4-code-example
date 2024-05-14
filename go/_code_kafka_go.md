```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	kafkaconn "github.com/pip-services4/pip-services4-go/pip-services4-kafka-go/connect"
)

func main() {
	kc := kafkaconn.NewKafkaConnection()
	config := conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 9092,
	)

	kc.Configure(context.Background(), config)

	kc.Open(context.Background())
}
```

```
ok := kc.IsOpen()
```

```
// kc.checkOpen() does not supports in curent version of Kafka golang toolkit 
```

```
err := kc.CreateQueue("my_queueA")
if err != nil {
	panic(err)
}
```

```
err = kc.DeleteQueue("my_queueC")
if err != nil {
	panic(err)
}
```

```
names, err := kc.ReadQueueNames()
```

```
// Seek operation does not supports in curent version of Kafka golang toolkit 
```

```
var listOfMessages []*kafka.ProducerMessage
for i, val := range []string{"message 1", "message 2"} {
	msg := &kafka.ProducerMessage{}
	msg.Key = kafka.StringEncoder(i)
	msg.Value = kafka.ByteEncoder(val)
	msg.Timestamp = time.Now()
	listOfMessages = append(listOfMessages, msg)
}
kc.Publish(context.Background(), "my_topicA", listOfMessages)

```

```
import (
	"fmt"
	"sync"

	kafka "github.com/Shopify/sarama"
)

type MyListener struct {
	ready chan bool
	Lock  sync.Mutex
}

func NewMyListener() *MyListener {
	c := &MyListener{}
	c.ready = make(chan bool)
	return c
}

// Setup is run at the beginning of a new session, before ConsumeClaim.
func (c *MyListener) Setup(kafka.ConsumerGroupSession) error {
	// Mark the consumer as ready
	c.ready <- true
	close(c.ready)
	return nil
}

// Cleanup is run at the end of a session, once all ConsumeClaim goroutines have exited
// but before the offsets are committed for the very last time.
func (c *MyListener) Cleanup(kafka.ConsumerGroupSession) error {
	return nil
}

// ConsumeClaim must start a consumer loop of ConsumerGroupClaim's Messages().
// Once the Messages() channel is closed, the Handler must finish its processing
// loop and exit.
func (c *MyListener) ConsumeClaim(session kafka.ConsumerGroupSession, claim kafka.ConsumerGroupClaim) error {
	for {
		// if len(c.readablePartitions) == 0 || slices.Contains(c.readablePartitions, claim.Partition()) {
		select {
		case msg := <-claim.Messages():
			if msg != nil {
				fmt.Printf("%s, %s", msg.Topic, msg.Value)
			}
		// Should return when `session.Context()` is done.
		// If not, will raise `ErrRebalanceInProgress` or `read tcp <ip>:<port>: i/o timeout` when kafka rebalance. see:
		// https://github.com/Shopify/sarama/issues/1192
		case <-session.Context().Done():
			return nil
		}
		// }
	}
}

func (c *MyListener) SetReady(chFlag chan bool) {
	c.Lock.Lock()
	defer c.Lock.Unlock()
	c.ready = chFlag
}

// Returns: channel with bool flag ready
func (c *MyListener) Ready() chan bool {
	return c.ready
}

myListener := NewMyListener()

kc.Subscribe(context.Background(), "my_topicA", "", kafka.NewConfig(), myListener)
```

```
err = kc.Unsubscribe(context.Background(), "my_topicA", "test", myListener)
```

```
_ = kc.Close(context.Background())
```

```
import (
	"context"
	"fmt"
	"sync"
	"time"

	kafka "github.com/Shopify/sarama"
	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	kafkaconn "github.com/pip-services4/pip-services4-go/pip-services4-kafka-go/connect"
)

func main() {
	ctx := context.Background()

	kc := kafkaconn.NewKafkaConnection()
	config := conf.NewConfigParamsFromTuples(
		"connection.host", "localhost",
		"connection.port", 9092,
	)

	kc.Configure(context.Background(), config)

	kc.Open(context.Background())

	err := kc.CreateQueue("my_queueA")
	if err != nil {
		panic(err)
	}

	err = kc.DeleteQueue("my_queueA")
	if err != nil {
		panic(err)
	}

	_, _ = kc.ReadQueueNames()

	var listOfMessages1 []*kafka.ProducerMessage
	var listOfMessages2 []*kafka.ProducerMessage
	var listOfMessages3 []*kafka.ProducerMessage

	for i, val := range []string{"message 1", "message 2"} {
		msg := &kafka.ProducerMessage{}
		msg.Key = kafka.StringEncoder(rune(i))
		msg.Value = kafka.ByteEncoder(val)
		msg.Timestamp = time.Now()
		listOfMessages1 = append(listOfMessages1, msg)
	}

	for i, val := range []string{"message 3", "message 4"} {
		msg := &kafka.ProducerMessage{}
		msg.Key = kafka.StringEncoder(rune(i))
		msg.Value = kafka.ByteEncoder(val)
		msg.Timestamp = time.Now()
		listOfMessages2 = append(listOfMessages2, msg)
	}

	for i, val := range []string{"message 5", "message 6"} {
		msg := &kafka.ProducerMessage{}
		msg.Key = kafka.StringEncoder(rune(i))
		msg.Value = kafka.ByteEncoder(val)
		msg.Timestamp = time.Now()
		listOfMessages3 = append(listOfMessages3, msg)
	}

	err = kc.Publish(ctx, "my_topicA", listOfMessages1)
	if err != nil {
		panic(err)
	}

	err = kc.Publish(ctx, "my_topicB", listOfMessages2)
	if err != nil {
		panic(err)
	}

	err = kc.Publish(ctx, "my_topicC", listOfMessages3)
	if err != nil {
		panic(err)
	}

	myListener := NewMyListener()

	err = kc.Subscribe(ctx, "my_topicA", "test", kafka.NewConfig(), myListener)
	if err != nil {
		panic(err)
	}

	err = kc.Subscribe(ctx, "my_topicB", "test", kafka.NewConfig(), myListener)
	if err != nil {
		panic(err)
	}

	err = kc.Subscribe(ctx, "my_topicC", "test", kafka.NewConfig(), myListener)
	if err != nil {
		panic(err)
	}

	<-time.After(1000 * time.Microsecond)

	err = kc.Close(ctx)
	if err != nil {
		panic(err)
	}

	fmt.Println("Execution completed!")

}

type MyListener struct {
	ready chan bool
	Lock  sync.Mutex
}

func NewMyListener() *MyListener {
	c := &MyListener{}
	c.ready = make(chan bool)
	return c
}

// Setup is run at the beginning of a new session, before ConsumeClaim.
func (c *MyListener) Setup(kafka.ConsumerGroupSession) error {
	// Mark the consumer as ready
	c.ready <- true
	close(c.ready)
	return nil
}

// Cleanup is run at the end of a session, once all ConsumeClaim goroutines have exited
// but before the offsets are committed for the very last time.
func (c *MyListener) Cleanup(kafka.ConsumerGroupSession) error {
	return nil
}

// ConsumeClaim must start a consumer loop of ConsumerGroupClaim's Messages().
// Once the Messages() channel is closed, the Handler must finish its processing
// loop and exit.
func (c *MyListener) ConsumeClaim(session kafka.ConsumerGroupSession, claim kafka.ConsumerGroupClaim) error {
	for {
		// if len(c.readablePartitions) == 0 || slices.Contains(c.readablePartitions, claim.Partition()) {
		select {
		case msg := <-claim.Messages():
			if msg != nil {
				fmt.Printf("%s, %s", msg.Topic, msg.Value)
			}
		// Should return when `session.Context()` is done.
		// If not, will raise `ErrRebalanceInProgress` or `read tcp <ip>:<port>: i/o timeout` when kafka rebalance. see:
		// https://github.com/Shopify/sarama/issues/1192
		case <-session.Context().Done():
			return nil
		}
		// }
	}
}

func (c *MyListener) SetReady(chFlag chan bool) {
	c.Lock.Lock()
	defer c.Lock.Unlock()
	c.ready = chFlag
}

// Returns: channel with bool flag ready
func (c *MyListener) Ready() chan bool {
	return c.ready
}

```

```
import (
	"context"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	kafkaqueue "github.com/pip-services4/pip-services4-go/pip-services4-kafka-go/queues"
)


ctx := context.Background()

queue := kafkaqueue.NewKafkaMessageQueue("")
queue.Configure(ctx, conf.NewConfigParamsFromTuples(
	"topic", "mytopic",
	"connection.protocol", "tcp",
	"connection.host", "localhost",
	"connection.port", 9092,
	"options.autosubscribe", true,
))

queue.Open(ctx)
```

```
import cqueues "github.com/pip-services4/pip-services4-go/pip-services4-messaging-go/queues"

err := queue.Send(ctx, cqueues.NewMessageEnvelope("123", "mymessage", []byte("ABC")))
```

```
message, err := queue.Receive(ctx, 10000)
```

```
envelope, err := queue.Peek(ctx)
```

```
err := queue.Complete(ctx, message)
```

```
err := queue.Close(ctx)
```

```
import (
	"context"
	"fmt"
	"sync"
	"time"

	conf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	kafkaqueue "github.com/pip-services4/pip-services4-go/pip-services4-kafka-go/queues"
	cqueues "github.com/pip-services4/pip-services4-go/pip-services4-messaging-go/queues"
)

func main() {
	ctx := context.Background()

	queue := kafkaqueue.NewKafkaMessageQueue("my_queue")
	queue.Configure(ctx, conf.NewConfigParamsFromTuples(
		// "topic", "my_queue",
		"connection.protocol", "tcp",
		"connection.host", "localhost",
		"connection.port", 9092,
		"options.autosubscribe", true,
	))

	err := queue.Open(ctx)
	if err != nil {
		panic(err)
	}

	messageReceiver := NewMyMessageReceiver()
	queue.BeginListen(ctx, messageReceiver)

	envelope1 := cqueues.NewMessageEnvelope("123", "Test", []byte("Test message"))
	err = queue.Send(ctx, envelope1)
	if err != nil {
		panic(err)
	}

	// wait message
	for i := 0; i < 15; i++ {
		if messageReceiver.MessageCount() > 0 {
			fmt.Println(messageReceiver.MessageCount())
			break
		}
		<-time.After(500 * time.Millisecond)
	}

	envelope2 := messageReceiver.Messages()[0]
	fmt.Println(envelope1.GetMessageAsString() == envelope2.GetMessageAsString())

	queue.EndListen(ctx)
	err = queue.Close(ctx)
	if err != nil {
		panic(err)
	}
}

type MyMessageReceiver struct {
	_messages []*cqueues.MessageEnvelope
	lock      sync.Mutex
}

func NewMyMessageReceiver() *MyMessageReceiver {
	c := &MyMessageReceiver{
		_messages: make([]*cqueues.MessageEnvelope, 0),
	}
	return c
}

func (c *MyMessageReceiver) Messages() []*cqueues.MessageEnvelope {
	c.lock.Lock()
	defer c.lock.Unlock()
	return c._messages
}

func (c *MyMessageReceiver) MessageCount() int {
	c.lock.Lock()
	defer c.lock.Unlock()
	return len(c._messages)
}

func (c *MyMessageReceiver) Clear(ctx context.Context, correlationId string) error {
	c.lock.Lock()
	defer c.lock.Unlock()
	c._messages = make([]*cqueues.MessageEnvelope, 0)
	return nil
}

func (c *MyMessageReceiver) ReceiveMessage(ctx context.Context, envelope *cqueues.MessageEnvelope, queue cqueues.IMessageQueue) (err error) {
	c.lock.Lock()
	defer c.lock.Unlock()
	c._messages = append(c._messages, envelope)
	return nil
}
```
