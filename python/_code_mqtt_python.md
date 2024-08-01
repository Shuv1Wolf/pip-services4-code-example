
```python
from pip_services4_mqtt.queues import MqttMessageQueue

```

```python
queue = MqttMessageQueue("mytopic")
```

```python
queue = MqttMessageQueue()
```

```python
from pip_services4_components.config import ConfigParams

queue.configure(ConfigParams.from_tuples(
    "topic", "mytopic",                 # set topic
    'connection.protocol', 'mqtt',
    "connection.host", "localhost",
    "connection.port", 1883,
    'options.qos', '1',                 # sets the qos level to 'At least once'
    'options.autosubscribe', True,      # autosubscription on the topic
    'options.serialize_envelope', True  # converts object into mosquitto values 
))
```

```python
queue.open(ctx)

```

```python
queue.close(None)
```

```python
from pip_services4_messaging.queues import MessageEnvelope

queue.send(ctx, MessageEnvelope(None, "mymessage", "ABC123"))

``` 

```python
message = queue.receive(ctx, 10000)
```

```python

message_text = message.get_message_as_string() # Returns 'ABC123'
```

```python
receivedPeek = queue.peek(None)

```

```python
message_text = receivedPeek.get_message_as_string () # Returns 'ABCD123'

```

```python
receivedPeekBatch = queue.peek_batch(None, 3)

```

```python
message_text = receivedPeekBatch[0].get_message_as_string () # Returns 'ABC123'

```

```python
result = queue.read_message_count()  # Returns 1

```

```python
queue.configure(ConfigParams.from_tuples(
     "topic", "mytopic", # set topic
    'connection.protocol', 'mqtt',
    "connection.host", "localhost",
    "connection.port", 1883,
    'options.autosubscribe', True, # autosubscription on the topic
    'options.serialize_envelope', True # converts object into musquitto values 
))


```

```python
# Pre-requisites
from pip_services4_mqtt.queues import MqttMessageQueue
from pip_services4_components.config import ConfigParams
from pip_services4_messaging.queues import MessageEnvelope

# Component creation and configuration
queue = MqttMessageQueue() 

queue.configure(ConfigParams.from_tuples(
     "topic", "mytopic", # set topic
    'connection.protocol', 'mqtt',
    "connection.host", "localhost",
    "connection.port", 1883,
    'options.autosubscribe', True, # autosubscription on the topic
    'options.serialize_envelope', True # converts object into musquitto values 
))

# Connection
queue.open(ctx)

# Send a message
queue.send(ctx, MessageEnvelope(None, "mymessage", "ABC1234"))

# Receive a message
message = queue.receive(ctx, 10000)
print(message.get_message_as_string ()) # Prints 'ABC1234'

# Close the connection
queue.close(None)

```