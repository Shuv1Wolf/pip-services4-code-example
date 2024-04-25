/service/IBeaconsService.go
```
package logic

import (
	"context"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type IBeaconsService interface {
	GetBeacons(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error)

	GetBeaconById(ctx context.Context, beaconId string) (data1.BeaconV1, error)

	GetBeaconByUdi(ctx context.Context, beaconId string) (data1.BeaconV1, error)

	CalculatePosition(ctx context.Context, siteId string, udis []string) (data1.GeoPointV1, error)

	CreateBeacon(ctx context.Context, beacon data1.BeaconV1) (data1.BeaconV1, error)

	UpdateBeacon(ctx context.Context, beacon data1.BeaconV1) (data1.BeaconV1, error)

	DeleteBeaconById(ctx context.Context, beaconId string) (data1.BeaconV1, error)
}

```

/service/BeaconsService.go
```
package logic

import (
	"context"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-data-go/keys"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type BeaconsService struct {
	persistence persist.IBeaconsPersistence
	commandSet  *BeaconsCommandSet
}

func NewBeaconsService() *BeaconsService {
	c := &BeaconsService{}
	return c
}

func (c *BeaconsService) Configure(ctx context.Context, config *cconf.ConfigParams) {
	// Read configuration parameters here...
}

func (c *BeaconsService) SetReferences(ctx context.Context, references cref.IReferences) {
	locator := cref.NewDescriptor("beacons", "persistence", "*", "*", "1.0")
	p, err := references.GetOneRequired(locator)
	if p != nil && err == nil {
		if _pers, ok := p.(persist.IBeaconsPersistence); ok {
			c.persistence = _pers
			return
		}
	}
	panic(cref.NewReferenceError(ctx, locator))
}

func (c *BeaconsService) GetCommandSet() *ccmd.CommandSet {
	if c.commandSet == nil {
		c.commandSet = NewBeaconsCommandSet(c)
	}
	return &c.commandSet.CommandSet
}

func (c *BeaconsService) GetBeacons(ctx context.Context,
	filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error) {
	return c.persistence.GetPageByFilter(ctx, filter, paging)
}

func (c *BeaconsService) GetBeaconById(ctx context.Context,
	beaconId string) (data1.BeaconV1, error) {

	return c.persistence.GetOneById(ctx, beaconId)
}

func (c *BeaconsService) GetBeaconByUdi(ctx context.Context,
	beaconId string) (data1.BeaconV1, error) {

	return c.persistence.GetOneByUdi(ctx, beaconId)
}

func (c *BeaconsService) CalculatePosition(ctx context.Context,
	siteId string, udis []string) (data1.GeoPointV1, error) {

	if udis == nil || len(udis) == 0 {
		return data1.GeoPointV1{}, nil
	}

	page, err := c.persistence.GetPageByFilter(
		ctx,

		*cquery.NewFilterParamsFromTuples(
			"site_id", siteId,
			"udis", udis,
		),
		*cquery.NewEmptyPagingParams(),
	)

	if err != nil || !page.HasData() {
		return data1.GeoPointV1{}, err
	}

	var lat float32 = 0
	var lng float32 = 0
	var count = 0

	for _, beacon := range page.Data {
		if beacon.Center.Type == "Point" {
			lng += beacon.Center.Coordinates[0]
			lat += beacon.Center.Coordinates[1]
			count += 1
		}
	}

	pos := data1.GeoPointV1{
		Type:        "Point",
		Coordinates: make([]float32, 2, 2),
	}

	if count > 0 {
		pos.Type = "Point"
		pos.Coordinates[0] = lng / (float32)(count)
		pos.Coordinates[1] = lat / (float32)(count)
	}

	return pos, nil
}

func (c *BeaconsService) CreateBeacon(ctx context.Context,
	beacon data1.BeaconV1) (data1.BeaconV1, error) {

	if beacon.Id == "" {
		beacon.Id = cdata.IdGenerator.NextLong()
	}

	if beacon.Type == "" {
		beacon.Type = data1.Unknown
	}

	return c.persistence.Create(ctx, beacon)
}

func (c *BeaconsService) UpdateBeacon(ctx context.Context,
	beacon data1.BeaconV1) (data1.BeaconV1, error) {

	if beacon.Type == "" {
		beacon.Type = data1.Unknown
	}

	return c.persistence.Update(ctx, beacon)
}

func (c *BeaconsService) DeleteBeaconById(ctx context.Context,
	beaconId string) (data1.BeaconV1, error) {

	return c.persistence.DeleteById(ctx, beaconId)
}

```

/service/BeaconsCommandSet.ts
```
package logic

import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	exec "github.com/pip-services4/pip-services4-go/pip-services4-components-go/exec"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
	ccmd "github.com/pip-services4/pip-services4-go/pip-services4-rpc-go/commands"
)

type BeaconsCommandSet struct {
	ccmd.CommandSet
	controller      IBeaconsService
	beaconConvertor cconv.IJSONEngine[data1.BeaconV1]
}

func NewBeaconsCommandSet(controller IBeaconsService) *BeaconsCommandSet {
	c := &BeaconsCommandSet{
		CommandSet:      *ccmd.NewCommandSet(),
		controller:      controller,
		beaconConvertor: cconv.NewDefaultCustomTypeJsonConvertor[data1.BeaconV1](),
	}

	c.AddCommand(c.makeGetBeaconsCommand())
	c.AddCommand(c.makeGetBeaconByIdCommand())
	c.AddCommand(c.makeGetBeaconByUdiCommand())
	c.AddCommand(c.makeCalculatePositionCommand())
	c.AddCommand(c.makeCreateBeaconCommand())
	c.AddCommand(c.makeUpdateBeaconCommand())
	c.AddCommand(c.makeDeleteBeaconByIdCommand())

	return c
}

func (c *BeaconsCommandSet) makeGetBeaconsCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"get_beacons",
		cvalid.NewObjectSchema().
			WithOptionalProperty("filter", cvalid.NewFilterParamsSchema()).
			WithOptionalProperty("paging", cvalid.NewPagingParamsSchema()),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			filter := cquery.NewEmptyFilterParams()
			paging := cquery.NewEmptyPagingParams()
			if _val, ok := args.Get("filter"); ok {
				filter = cquery.NewFilterParamsFromValue(_val)
			}
			if _val, ok := args.Get("paging"); ok {
				paging = cquery.NewPagingParamsFromValue(_val)
			}
			return c.controller.GetBeacons(ctx, *filter, *paging)
		})
}

func (c *BeaconsCommandSet) makeGetBeaconByIdCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"get_beacon_by_id",
		cvalid.NewObjectSchema().
			WithRequiredProperty("beacon_id", cconv.String),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			return c.controller.GetBeaconById(ctx, args.GetAsString("beacon_id"))
		})
}

func (c *BeaconsCommandSet) makeGetBeaconByUdiCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"get_beacon_by_udi",
		cvalid.NewObjectSchema().
			WithRequiredProperty("udi", cconv.String),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			return c.controller.GetBeaconByUdi(ctx, args.GetAsString("udi"))
		})
}

func (c *BeaconsCommandSet) makeCalculatePositionCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"calculate_position",
		cvalid.NewObjectSchema().
			WithRequiredProperty("site_id", cconv.String).
			WithRequiredProperty("udis", cvalid.NewArraySchema(cconv.String)),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			return c.controller.CalculatePosition(
				ctx,

				args.GetAsString("site_id"),
				strings.Split(args.GetAsString("udis"), ","),
			)
		})
}

func (c *BeaconsCommandSet) makeCreateBeaconCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"create_beacon",
		cvalid.NewObjectSchema().
			WithRequiredProperty("beacon", data1.NewBeaconV1Schema()),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {

			var beacon data1.BeaconV1
			if _beacon, ok := args.GetAsObject("beacon"); ok {
				buf, err := cconv.JsonConverter.ToJson(_beacon)
				if err != nil {
					return nil, err
				}
				beacon, err = c.beaconConvertor.FromJson(buf)
				if err != nil {
					return nil, err
				}
			}
			return c.controller.CreateBeacon(ctx, beacon)
		})
}

func (c *BeaconsCommandSet) makeUpdateBeaconCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"update_beacon",
		cvalid.NewObjectSchema().
			WithRequiredProperty("beacon", data1.NewBeaconV1Schema()),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			var beacon data1.BeaconV1
			if _beacon, ok := args.GetAsObject("beacon"); ok {
				buf, err := cconv.JsonConverter.ToJson(_beacon)
				if err != nil {
					return nil, err
				}
				beacon, err = c.beaconConvertor.FromJson(buf)
				if err != nil {
					return nil, err
				}
			}
			return c.controller.UpdateBeacon(ctx, beacon)
		})
}

func (c *BeaconsCommandSet) makeDeleteBeaconByIdCommand() ccmd.ICommand {
	return ccmd.NewCommand(
		"delete_beacon_by_id",
		cvalid.NewObjectSchema().
			WithRequiredProperty("beacon_id", cconv.String),
		func(ctx context.Context, args *exec.Parameters) (result any, err error) {
			return c.controller.DeleteBeaconById(ctx, args.GetAsString("beacon_id"))
		})
}

```

/test/service/BeaconsService_test.go
```
package test_logic

import (
	"context"
	"testing"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	logic "github.com/pip-services-samples/service-beacons-go/service"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	"github.com/stretchr/testify/assert"
)

type BeaconsServiceTest struct {
	BEACON1     *data1.BeaconV1
	BEACON2     *data1.BeaconV1
	persistence *persist.BeaconsMemoryPersistence
	service     *logic.BeaconsService
}

func newBeaconsServiceTest() *BeaconsServiceTest {
	BEACON1 := &data1.BeaconV1{
		Id:     "1",
		Udi:    "00001",
		Type:   data1.AltBeacon,
		SiteId: "1",
		Label:  "TestBeacon1",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: []float32{0.0, 0.0}},
		Radius: 50,
	}

	BEACON2 := &data1.BeaconV1{
		Id:     "2",
		Udi:    "00002",
		Type:   data1.IBeacon,
		SiteId: "1",
		Label:  "TestBeacon2",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: []float32{2.0, 2.0}},
		Radius: 70,
	}

	persistence := persist.NewBeaconsMemoryPersistence()
	persistence.Configure(context.Background(), cconf.NewEmptyConfigParams())

	service := logic.NewBeaconsService()
	service.Configure(context.Background(), cconf.NewEmptyConfigParams())

	references := cref.NewReferencesFromTuples(
		context.Background(),
		cref.NewDescriptor("beacons", "persistence", "memory", "default", "1.0"), persistence,
		cref.NewDescriptor("beacons", "service", "default", "default", "1.0"), service,
	)

	service.SetReferences(context.Background(), references)

	return &BeaconsServiceTest{
		BEACON1:     BEACON1,
		BEACON2:     BEACON2,
		persistence: persistence,
		service:     service,
	}
}

func (c *BeaconsServiceTest) setup(t *testing.T) {
	err := c.persistence.Open(context.Background())
	if err != nil {
		t.Error("Failed to open persistence", err)
	}

	err = c.persistence.Clear(context.Background())
	if err != nil {
		t.Error("Failed to clear persistence", err)
	}
}

func (c *BeaconsServiceTest) teardown(t *testing.T) {
	err := c.persistence.Close(context.Background())
	if err != nil {
		t.Error("Failed to close persistence", err)
	}
}

func (c *BeaconsServiceTest) testCrudOperations(t *testing.T) {
	var beacon1 data1.BeaconV1

	// Create the first beacon
	beacon, err := c.service.CreateBeacon(context.Background(), c.BEACON1.Clone())
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON1.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON1.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON1.Type, beacon.Type)
	assert.Equal(t, c.BEACON1.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Create the second beacon
	beacon, err = c.service.CreateBeacon(context.Background(), c.BEACON2.Clone())
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON2.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON2.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON2.Type, beacon.Type)
	assert.Equal(t, c.BEACON2.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Get all beacons
	page, err := c.service.GetBeacons(context.Background(), *cquery.NewEmptyFilterParams(), *cquery.NewEmptyPagingParams())
	assert.Nil(t, err)
	assert.NotNil(t, page)
	assert.True(t, page.HasData())
	assert.Len(t, page.Data, 2)
	beacon1 = page.Data[0].Clone()

	// Update the beacon
	beacon1.Label = "ABC"
	beacon, err = c.service.UpdateBeacon(context.Background(), beacon1)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)
	assert.Equal(t, "ABC", beacon.Label)

	// Get beacon by udi
	beacon, err = c.service.GetBeaconByUdi(context.Background(), beacon1.Udi)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Delete the beacon
	beacon, err = c.service.DeleteBeaconById(context.Background(), beacon1.Id)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Try to get deleted beacon
	beacon, err = c.service.GetBeaconById(context.Background(), beacon1.Id)
	assert.Nil(t, err)
	assert.Equal(t, data1.BeaconV1{}, beacon)
}

func (c *BeaconsServiceTest) testCalculatePositions(t *testing.T) {
	// Create the first beacon
	beacon, err := c.service.CreateBeacon(context.Background(), c.BEACON1.Clone())
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON1.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON1.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON1.Type, beacon.Type)
	assert.Equal(t, c.BEACON1.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Create the second beacon
	beacon, err = c.service.CreateBeacon(context.Background(), c.BEACON2.Clone())
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON2.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON2.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON2.Type, beacon.Type)
	assert.Equal(t, c.BEACON2.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Calculate position for one beacon
	position, err := c.service.CalculatePosition(context.Background(), "1", []string{"00001"})
	assert.Nil(t, err)
	assert.NotEqual(t, data1.GeoPointV1{}, position)
	assert.Equal(t, "Point", position.Type)
	assert.Equal(t, (float32)(0.0), position.Coordinates[0])
	assert.Equal(t, (float32)(0.0), position.Coordinates[1])

	// Calculate position for two beacons
	position, err = c.service.CalculatePosition(context.Background(), "1", []string{"00001", "00002"})
	assert.Nil(t, err)
	assert.NotEqual(t, data1.GeoPointV1{}, position)
	assert.Equal(t, "Point", position.Type)
	assert.Equal(t, (float32)(1.0), position.Coordinates[0])
	assert.Equal(t, (float32)(1.0), position.Coordinates[1])
}

func TestBeaconsService(t *testing.T) {
	c := newBeaconsServiceTest()

	c.setup(t)
	t.Run("CRUD Operations", c.testCrudOperations)
	c.teardown(t)

	c.setup(t)
	t.Run("Calculate Positions", c.testCalculatePositions)
	c.teardown(t)
}

```