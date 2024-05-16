```
version: '3.3'
services:
  app:  
    image: "pipservices/service-beacons-go:latest"  
    ports:  
      - "8080:8080"  
    depends_on:  
      - mongo   
    environment:  
      - HTTP_ENABLED=true  
      - HTTP_PORT=8080  
      - MONGO_ENABLED=true  
      - MONGO_SERVICE_HOST=mongo  
      - MONGO_SERVICE_PORT=27017
  mongo:  
    image: mongo:latest  
    ports:    
      - "27017:27017"
```

```
package test_clients1

import (
	"context"
	"testing"

	clients1 "github.com/pip-services-samples/client-beacons-go/clients/version1"
	controller1 "github.com/pip-services-samples/service-beacons-go/controllers/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	logic "github.com/pip-services-samples/service-beacons-go/service"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

type beaconsCommandableHttpClientV1Test struct {
	persistence *persist.BeaconsMemoryPersistence
	service     *logic.BeaconsService
	controller  *controller1.BeaconsHttpControllerV1
	client      *clients1.BeaconsHttpClientV1
	fixture     *BeaconsClientV1Fixture
	ctx         context.Context
}

func newBeaconsHttpClientV1Test() *beaconsCommandableHttpClientV1Test {
	ctx := context.Background()
	persistence := persist.NewBeaconsMemoryPersistence()
	persistence.Configure(ctx, cconf.NewEmptyConfigParams())

	service := logic.NewBeaconsService()
	service.Configure(ctx, cconf.NewEmptyConfigParams())

	httpConfig := cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.port", "3000",
		"connection.host", "localhost",
	)

	controller := controller1.NewBeaconsHttpControllerV1()
	controller.Configure(ctx, httpConfig)

	client := clients1.NewBeaconsHttpClientV1()
	client.Configure(ctx, httpConfig)

	references := cref.NewReferencesFromTuples(ctx,
		cref.NewDescriptor("beacons", "persistence", "memory", "default", "1.0"), persistence,
		cref.NewDescriptor("beacons", "service", "default", "default", "1.0"), service,
		cref.NewDescriptor("beacons", "controller", "http", "default", "1.0"), controller,
		cref.NewDescriptor("beacons", "client", "http", "default", "1.0"), client,
	)
	service.SetReferences(ctx, references)
	controller.SetReferences(ctx, references)
	client.SetReferences(ctx, references)

	fixture := NewBeaconsClientV1Fixture(client)

	return &beaconsCommandableHttpClientV1Test{
		persistence: persistence,
		controller:  controller,
		service:     service,
		client:      client,
		fixture:     fixture,
		ctx:         ctx,
	}
}

func (c *beaconsCommandableHttpClientV1Test) setup(t *testing.T) {
	err := c.persistence.Open(c.ctx)
	if err != nil {
		t.Error("Failed to open persistence", err)
	}

	err = c.controller.Open(c.ctx)
	if err != nil {
		t.Error("Failed to open service", err)
	}

	err = c.client.Open(c.ctx)
	if err != nil {
		t.Error("Failed to open client", err)
	}

	err = c.persistence.Clear(c.ctx)
	if err != nil {
		t.Error("Failed to clear persistence", err)
	}
}

func (c *beaconsCommandableHttpClientV1Test) teardown(t *testing.T) {
	err := c.client.Close(c.ctx)
	if err != nil {
		t.Error("Failed to close client", err)
	}

	err = c.controller.Close(c.ctx)
	if err != nil {
		t.Error("Failed to close service", err)
	}

	err = c.persistence.Close(c.ctx)
	if err != nil {
		t.Error("Failed to close persistence", err)
	}
}

func TestBeaconsHttpClientV1(t *testing.T) {
	c := newBeaconsHttpClientV1Test()

	c.setup(t)

	t.Run("CRUD Operations", c.fixture.TestCrudOperations)
	c.teardown(t)

	c.setup(t)

	t.Run("Calculate Positions", c.fixture.TestCalculatePosition)
	c.teardown(t)
}

```

**/test/version1/BeaconsClientV1Fixture.go**

```
package test_clients1

import (
	"context"
	"testing"

	clients1 "github.com/pip-services-samples/client-beacons-go/clients/version1"
	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	"github.com/stretchr/testify/assert"
)

type BeaconsClientV1Fixture struct {
	BEACON1 *data1.BeaconV1
	BEACON2 *data1.BeaconV1
	client  clients1.IBeaconsClientV1
	ctx     context.Context
}

func NewBeaconsClientV1Fixture(client clients1.IBeaconsClientV1) *BeaconsClientV1Fixture {
	c := &BeaconsClientV1Fixture{}

	c.BEACON1 = &data1.BeaconV1{
		Id:     "1",
		Udi:    "00001",
		Type:   data1.AltBeacon,
		SiteId: "1",
		Label:  "TestBeacon1",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: []float32{0.0, 0.0}},
		Radius: 50,
	}

	c.BEACON2 = &data1.BeaconV1{
		Id:     "2",
		Udi:    "00002",
		Type:   data1.IBeacon,
		SiteId: "1",
		Label:  "TestBeacon2",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: []float32{2.0, 2.0}},
		Radius: 70,
	}

	c.client = client
	c.ctx = context.Background()

	return c
}

func (c *BeaconsClientV1Fixture) testCreateBeacons(t *testing.T) {
	// Create the first beacon
	beacon, err := c.client.CreateBeacon(c.ctx, *c.BEACON1)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, c.BEACON1.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON1.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON1.Type, beacon.Type)
	assert.Equal(t, c.BEACON1.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Create the second beacon
	beacon, err = c.client.CreateBeacon(c.ctx, *c.BEACON2)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, c.BEACON2.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON2.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON2.Type, beacon.Type)
	assert.Equal(t, c.BEACON2.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)
}

func (c *BeaconsClientV1Fixture) TestCrudOperations(t *testing.T) {
	var beacon1 *data1.BeaconV1

	// Create items
	c.testCreateBeacons(t)

	// Get all beacons
	page, err := c.client.GetBeacons(c.ctx, *cquery.NewEmptyFilterParams(), *cquery.NewEmptyPagingParams())
	assert.Nil(t, err)
	assert.NotNil(t, page)
	assert.Len(t, page.Data, 2)
	beacon1 = &page.Data[0]

	// Update the beacon
	beacon1.Label = "ABC"
	beacon, err := c.client.UpdateBeacon(c.ctx, *beacon1)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)
	assert.Equal(t, "ABC", beacon.Label)

	// Get beacon by udi
	beacon, err = c.client.GetBeaconByUdi(c.ctx, beacon1.Udi)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Delete the beacon
	beacon, err = c.client.DeleteBeaconById(c.ctx, beacon1.Id)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Try to get deleted beacon
	beacon, err = c.client.GetBeaconById(c.ctx, beacon1.Id)
	assert.Nil(t, err)
	assert.Empty(t, beacon)
}

func (c *BeaconsClientV1Fixture) TestCalculatePosition(t *testing.T) {
	// Create items
	c.testCreateBeacons(t)

	// Calculate position for one beacon
	position, err := c.client.CalculatePosition(c.ctx, "1", []string{"00001"})
	assert.Nil(t, err)
	assert.NotNil(t, position)
	assert.Equal(t, "Point", position.Type)
	assert.Equal(t, (float32)(0), position.Coordinates[0])
	assert.Equal(t, (float32)(0), position.Coordinates[1])
}

```

```
docker-compose -f ./docker/docker-compose.dev.yml up 
```

```
go test -v ./test/...
```

```

[beacons:INFO:2020-06-24T15:27:42.747Z] Press Control-C to stop the microservice...
[beacons:DEBUG:2020-06-24T15:27:42.982Z] Opened REST service at http://0.0.0.0:8080
[beacons:INFO:2020-06-24T15:27:42.983Z] Container beacons started.
[123:TRACE:2020-06-24T15:31:19.003Z] Executing v1/beacons.create_beacon method
[123:TRACE:2020-06-24T15:31:19.007Z] Created item 1
[123:TRACE:2020-06-24T15:31:19.093Z] Executing v1/beacons.create_beacon method
[123:TRACE:2020-06-24T15:31:19.093Z] Created item 2
[123:TRACE:2020-06-24T15:31:19.108Z] Executing v1/beacons.get_beacons method
[123:TRACE:2020-06-24T15:31:19.111Z] Retrieved 2 items
[123:TRACE:2020-06-24T15:31:19.165Z] Executing v1/beacons.update_beacon method
[123:TRACE:2020-06-24T15:31:19.167Z] Updated item 1
[123:TRACE:2020-06-24T15:31:19.218Z] Executing v1/beacons.get_beacon_by_udi method
[123:TRACE:2020-06-24T15:31:19.220Z] Found beacon by 00001
[123:TRACE:2020-06-24T15:31:19.270Z] Executing v1/beacons.delete_beacon_by_id method
[123:TRACE:2020-06-24T15:31:19.271Z] Deleted item by 1
[123:TRACE:2020-06-24T15:31:19.322Z] Executing v1/beacons.get_beacon_by_id method
[123:TRACE:2020-06-24T15:31:19.322Z] Cannot find item by 1
[123:TRACE:2020-06-24T15:31:19.332Z] Executing v1/beacons.delete_beacon_by_id method
[123:TRACE:2020-06-24T15:31:19.333Z] Deleted item by 2
[123:TRACE:2020-06-24T15:31:21.435Z] Executing v1/beacons.create_beacon method
[123:TRACE:2020-06-24T15:31:21.435Z] Created item 1
[123:TRACE:2020-06-24T15:31:21.448Z] Executing v1/beacons.create_beacon method
[123:TRACE:2020-06-24T15:31:21.449Z] Created item 2
[123:TRACE:2020-06-24T15:31:21.505Z] Executing v1/beacons.calculate_position method
[123:TRACE:2020-06-24T15:31:21.509Z] Retrieved 1 items
[123:TRACE:2020-06-24T15:31:21.562Z] Executing v1/beacons.calculate_position method
[123:TRACE:2020-06-24T15:31:21.563Z] Retrieved 2 items
```
