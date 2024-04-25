/bin/main.go

```
package main

import (
	"context"
	"os"

	cont "github.com/pip-services-samples/service-beacons-go/containers"
)

func main() {
	proc := cont.NewBeaconsProcess()
	proc.SetConfigPath("./config/config.yml")
	proc.Run(context.Background(), os.Args)
}

```

```
go run ./bin/main.go
```

```
export MONGO_ENABLED=true

go run ./bin/main.go
```

**/docker/docker-compose.yml**

```
version: '3.3'

services:

  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"


```

```
docker-compose -f ./docker/docker-compose.yml up
```

```
{
  "components": [
    "pip-services:context-info:default:default:1.0",
    "pip-services:factory:container:default:1.0",
    "pip-services:context-info:default:default:1.0",
    "pip-services:logger:console:default:1.0",
    "pip-services:tracer:log:default:1.0",
    "pip-services:counters:log:default:1.0",
    "beacons:persistence:memory:default:1.0",
    "beacons:service:default:default:1.0",
    "pip-services:endpoint:http:default:1.0",
    "beacons:controller:http:default:1.0",
    "pip-services:heartbeat-controller:http:default:1.0",
    "pip-services:status-controller:http:default:1.0"
  ],
  "current_time": "2024-04-25T14:57:38+03:00",
  "description": "Beacons microservice",
  "id": "microservice-server",
  "name": "beacons",
  "properties": {

  },
  "start_time": "2024-04-25T14:52:45+03:00",
  "uptime": 292457319100
}
```

```
curl -X POST -H "Content-Type: application/json" -d "{\"beacon\":{\"id\": \"1\", \"udi\": \"00001\",  \"type\": \"altbeacon\",  \"site_id\": \"1\", \"label\": \"TestBeacon1\", \"center\": { \"type\": \"Point\", \"coordinates\": [ 0, 0 ] },\"radius\": 50}}" http://localhost:8080/v1/beacons/create_beacon
‍
curl -X POST -H "Content-Type: application/json" -d "{\"beacon\":{\"id\": \"2\", \"udi\": \"00002\",  \"type\": \"altbeacon\",  \"site_id\": \"1\",    \"label\": \"TestBeacon2\", \"center\": { \"type\": \"Point\", \"coordinates\": [ 2, 2 ] },\"radius\": 70}}" http://localhost:8080/v1/beacons/create_beacon


```

```
curl -X POST -H "Content-Type: application/json" -d "{\"beacon\":{\"id\": \"2\", \"udi\": \"00002\", \"type\": \"altbeacon\", \"site_id\": \"1\", \"label\": \"TestBeacon2\", \"center\": { \"type\": \"Point\", \"coordinates\": [ 2, 4 ] },\"radius\": 80}}" http://localhost:8080/v1/beacons/update_beacon


```

```
curl -X POST http://localhost:8080/v1/beacons/get_beacons
```

```
{
  "total":2,
  "data":[
    {
      "id":"1",
      "udi":"00001",
      "type":"altbeacon",
      "site_id":"1",
      "label":"TestBeacon1",
      "center":{
        "type":"Point",
        "coordinates":[
          0,
          0
        ]
      },
      "radius":50
    },
    {
      "id":"2",
      "udi":"00002",
      "type":"altbeacon",
      "site_id":"1",
      "label":"TestBeacon2",
      "center":{
        "type":"Point",
        "coordinates":[
          2,
          4
        ]
      },
      "radius":80
    }
  ]
}
```

```
curl -X POST -H "Content-Type: application/json" -d "{\"udi\":\"00001\"}" http://localhost:8080/v1/beacons/get_beacon_by_udi

```

```
{
  "id":"1",
  "udi":"00001",
  "type":"altbeacon",
  "site_id":"1",
  "label":"TestBeacon1",
  "center":{
    "type":"Point",
    "coordinates":[
      0,
      0
    ]
  },
  "radius":50
}
```

```
curl -X POST -H "Content-Type: application/json" -d "{\"site_id\": \"1\",\"udis\": [\"00001\", \"00002\"]}" http://localhost:8080/v1/beacons/calculate_position

```

```
{"type":"Point","coordinates":[1,2]}
```

```
curl -X POST -H "Content-Type: application/json" -d "{\"beacon_id\":\"1\"}" http://localhost:8080/v1/beacons/delete_beacon_by_id
```

```

{
  "id":"1",
  "udi":"00001",
  "type":"altbeacon",
  "site_id":"1",
  "label":"TestBeacon1",
  "center":{
    "type":"Point",
    "coordinates":[
      0,
      0
    ]
  },
  "radius":50
}
```

```
curl -X POST  http://localhost:8080/v1/beacons/get_beacons

```

```
{
  "total":1,
  "data":[
    {
      "id":"2",
      "udi":"00002",
      "type":"altbeacon",
      "site_id":"1",
      "label":"TestBeacon2",
      "center":{
        "type":"Point",
        "coordinates":[
          2,
          4
        ]
      },
      "radius":80
    }
  ]
}

```
