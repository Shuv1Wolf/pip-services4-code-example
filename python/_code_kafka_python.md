
```python
from pip_services4_kafka.connect import KafkaConnection
from pip_services4_components.config import ConfigParams

kc = KafkaConnection()
config = ConfigParams.from_tuples(
    'connection.host', 'localhost', 
    'connection.port', 9092
)

kc.configure(config)
kc.open(None)
```

```python
kc.is_open()
```

```python
kc._check_open()
```

```python
kc.create_queue("my_queueA")
```

```python
kc.delete_queue('my_queueC')
```

```python
kc.read_queue_names()
```

```python
offset = 60
kc.seek("my_topicA", None, 0, offset, my_listener)
```

```python
list_of_messages = [{'value': "message 1"}, {'value': "message 2"}]
kc.publish("my_topicA", list_of_messages, {})
```

```python
from pip_services4_kafka.connect import IKafkaMessageListener

class MyListener(IKafkaMessageListener):
    
    def on_message(topic, partition, message):
        print(message.value().decode('utf-8'), message.offset())

my_listener = MyListener()
kc.subscribe("my_topicA", None, {}, my_listener)
```

```python
kc.unsubscribe("my_topicA", None, my_listener)
```

```python
kc.close(None)
```

```python
from pip_services4_kafka.connect import KafkaConnection
from pip_services4_components.config import ConfigParams
from pip_services4_kafka.connect import IKafkaMessageListener

kc = KafkaConnection()
config = ConfigParams.from_tuples(
    'connection.host', 'localhost',
    'connection.port', 9092
)

kc.configure(config)
kc.open(None)

kc.create_queue("my_queueA")
kc.create_queue("my_queueB")
kc.create_queue("my_queueC")

list_of_messages1 = [{'value': "message 1"}, {'value': "message 2"}]
list_of_messages2 = [{'value': "message 3"}, {'value': "message 4"}]
list_of_messages3 = [{'value': "message 5"}, {'value': "message 6"}]

kc.publish("my_topicA", list_of_messages1, {})
kc.publish("my_topicB", list_of_messages2, {})
kc.publish("my_topicC", list_of_messages3, {})

class MyListener(IKafkaMessageListener):
    def on_message(self, topic, partition, message):
        print(message.value().decode('utf-8'), message.offset())

my_listener = MyListener()

kc.subscribe("my_topicA", None, {}, my_listener)
kc.subscribe("my_topicB", None, {}, my_listener)
kc.subscribe("my_topicC", None, {}, my_listener)

kc.close(None)

print("Execution completed!")
```

```python

from pip_services4_kafka.queues import KafkaMessageQueue

queue = KafkaMessageQueue()
queue.configure(ConfigParams.from_tuples(
    "topic", "mytopic",
    'connection.protocol', 'tcp',
    "connection.host", "localhost",
    "connection.port", 9092,
    "options.autosubscribe", True
))

queue.open(None)
```

```python

from pip_services4_messaging.queues import MessageEnvelope

queue.send(None, MessageEnvelope(None, "mymessage", "ABC"))
```

```python
message = queue.receive(None, 10000)
```

```python
envelope = queue.peek(None)
```

```python
queue.complete(message)
```

```python
queue.close(None)
```

```python
import threading
import time
from typing import List, Optional
from pip_services4_components.config import ConfigParams
from pip_services4_components.run import ICleanable
from pip_services4_messaging.queues import IMessageReceiver, MessageEnvelope, IMessageQueue
from pip_services4_kafka.queues import KafkaMessageQueue

class MyMessageReceiver(IMessageReceiver, ICleanable):

    def __init__(self):
        self.__messages: List[MessageEnvelope] = []
        self.__lock = threading.Lock()

    @property
    def messages(self) -> List[MessageEnvelope]:
        return self.__messages

    @property
    def message_count(self) -> int:
        return len(self.__messages)

    def receive_message(self, message: MessageEnvelope, queue: IMessageQueue):
        with self.__lock:
            self.__messages.append(message)

    def clear(self, correlation_id: Optional[str]):
        with self.__lock:
            self.__messages = []
                      
queue = KafkaMessageQueue()
queue.configure(ConfigParams.from_tuples(
    "topic", "mytopic2",
    'connection.protocol', 'tcp',
    "connection.host", "localhost",
    "connection.port", 9092,
    "options.autosubscribe", True
))
queue.open(None)

message_receiver = MyMessageReceiver()
queue.begin_listen(None, message_receiver)

envelope1 = MessageEnvelope(ctx, "Test", "Test message")
queue.send(None, envelope1)

# await message
for i in range(15):
    if len(message_receiver.messages) > 0:
        break
    time.sleep(0.5)

envelope2 = message_receiver.messages[0]

print(envelope1.message.decode('utf-8') == envelope2.message.decode('utf-8'))

queue.end_listen(None)

queue.close(None)

```