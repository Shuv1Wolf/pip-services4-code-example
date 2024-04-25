/persistence/IBeaconsPersistence.go
```
package persistence

import (
	"context"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
)

type IBeaconsPersistence interface {
	GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error)

	GetOneById(ctx context.Context, id string) (data1.BeaconV1, error)

	GetOneByUdi(ctx context.Context, udi string) (data1.BeaconV1, error)

	Create(ctx context.Context, item data1.BeaconV1) (data1.BeaconV1, error)

	Update(ctx context.Context, item data1.BeaconV1) (data1.BeaconV1, error)

	DeleteById(ctx context.Context, id string) (data1.BeaconV1, error)
}

```

/persistence/BeaconsMemoryPersistence.go
```
package persistence

import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cpersist "github.com/pip-services4/pip-services4-go/pip-services4-persistence-go/persistence"
)

type BeaconsMemoryPersistence struct {
	cpersist.IdentifiableMemoryPersistence[data1.BeaconV1, string]
}

func NewBeaconsMemoryPersistence() *BeaconsMemoryPersistence {
	c := &BeaconsMemoryPersistence{
		IdentifiableMemoryPersistence: *cpersist.NewIdentifiableMemoryPersistence[data1.BeaconV1, string](),
	}
	c.IdentifiableMemoryPersistence.MaxPageSize = 1000
	return c
}

func (c *BeaconsMemoryPersistence) composeFilter(filter cquery.FilterParams) func(beacon data1.BeaconV1) bool {

	id := filter.GetAsString("id")
	siteId := filter.GetAsString("site_id")
	label := filter.GetAsString("label")
	udi := filter.GetAsString("udi")

	var udiValues []string
	if _udis, ok := filter.GetAsObject("udis"); ok {
		if _val, ok := _udis.([]string); ok {
			udiValues = _val
		}
		if _val, ok := _udis.(string); ok {
			udiValues = strings.Split(_val, ",")
		}

	}

	return func(beacon data1.BeaconV1) bool {
		if id != "" && beacon.Id != id {
			return false
		}
		if siteId != "" && beacon.SiteId != siteId {
			return false
		}
		if label != "" && beacon.Label != label {
			return false
		}
		if udi != "" && beacon.Udi != udi {
			return false
		}
		if len(udiValues) > 0 && !ContainsStr(udiValues, beacon.Udi) {
			return false
		}
		return true
	}
}

func (c *BeaconsMemoryPersistence) GetPageByFilter(ctx context.Context,
	filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error) {

	return c.IdentifiableMemoryPersistence.
		GetPageByFilter(ctx, c.composeFilter(filter), paging, nil, nil)
}

func (c *BeaconsMemoryPersistence) GetOneByUdi(ctx context.Context, udi string) (data1.BeaconV1, error) {
	var item *data1.BeaconV1
	for _, beacon := range c.IdentifiableMemoryPersistence.Items {
		if beacon.Udi == udi {
			_item := beacon.Clone()
			item = &_item
			break
		}
	}

	if item != nil {
		c.IdentifiableMemoryPersistence.Logger.Trace(ctx, "Found beacon by %s", udi)
	} else {
		c.IdentifiableMemoryPersistence.Logger.Trace(ctx, "Cannot find beacon by %s", udi)
	}

	return *item, nil
}

func ContainsStr(arr []string, substr string) bool {
	for _, _str := range arr {
		if _str == substr {
			return true
		}
	}
	return false
}

```

/persistence/BeaconsMongoPersistence.go
```
package persistence

import (
	"context"
	"strings"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	cmongo "github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go/persistence"
	"go.mongodb.org/mongo-driver/bson"
)

type BeaconsMongoPersistence struct {
	cmongo.IdentifiableMongoDbPersistence[data1.BeaconV1, string]
}

func NewBeaconsMongoPersistence() *BeaconsMongoPersistence {
	c := &BeaconsMongoPersistence{}
	c.IdentifiableMongoDbPersistence = *cmongo.InheritIdentifiableMongoDbPersistence[data1.BeaconV1, string](c, "beacons")
	return c
}

func (c *BeaconsMongoPersistence) GetPageByFilter(ctx context.Context, filter cquery.FilterParams, paging cquery.PagingParams) (cquery.DataPage[data1.BeaconV1], error) {
	filterObj := bson.M{}
	if id, ok := filter.GetAsNullableString("id"); ok && id != "" {
		filterObj["_id"] = id
	}
	if siteId, ok := filter.GetAsNullableString("site_id"); ok && siteId != "" {
		filterObj["site_id"] = siteId
	}
	if typeId, ok := filter.GetAsNullableString("type"); ok && typeId != "" {
		filterObj["type"] = typeId
	}
	if udi, ok := filter.GetAsNullableString("udi"); ok && udi != "" {
		filterObj["udi"] = udi
	}
	if label, ok := filter.GetAsNullableString("label"); ok && label != "" {
		filterObj["label"] = label
	}
	if udis, ok := filter.GetAsObject("udis"); ok {
		var udisM bson.M
		switch _udis := udis.(type) {
		case []string:
			if len(_udis) > 0 {
				udisM = bson.M{"$in": _udis}
			}
			break
		case string:
			if _udisArr := strings.Split(_udis, ","); len(_udisArr) > 0 {
				udisM = bson.M{"$in": _udisArr}
			}
			break
		}
		if udisM != nil {
			filterObj["udi"] = udisM
		}
	}

	return c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx,
		filterObj, paging,
		nil, nil,
	)
}

func (c *BeaconsMongoPersistence) GetOneByUdi(ctx context.Context, udi string) (data1.BeaconV1, error) {

	paging := *cquery.NewPagingParams(0, 1, false)
	page, err := c.IdentifiableMongoDbPersistence.GetPageByFilter(ctx,
		bson.M{"udi": udi}, paging,
		nil, nil,
	)
	if err != nil {
		return data1.BeaconV1{}, err
	}
	if page.HasData() {
		return page.Data[0], nil
	}
	return data1.BeaconV1{}, nil
}

```

/test/persistence/BeaconsPersistenceFixture.go
```
package test_persistence

import (
	"context"
	"testing"

	data1 "github.com/pip-services-samples/service-beacons-go/data/version1"
	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	cquery "github.com/pip-services4/pip-services4-go/pip-services4-data-go/query"
	"github.com/stretchr/testify/assert"
)

type BeaconsPersistenceFixture struct {
	BEACON1     *data1.BeaconV1
	BEACON2     *data1.BeaconV1
	BEACON3     *data1.BeaconV1
	persistence persist.IBeaconsPersistence
}

func NewBeaconsPersistenceFixture(persistence persist.IBeaconsPersistence) *BeaconsPersistenceFixture {
	c := BeaconsPersistenceFixture{}

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

	c.BEACON3 = &data1.BeaconV1{
		Id:     "3",
		Udi:    "00003",
		Type:   data1.AltBeacon,
		SiteId: "2",
		Label:  "TestBeacon3",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: []float32{10.0, 10.0}},
		Radius: 50,
	}

	c.persistence = persistence
	return &c
}

func (c *BeaconsPersistenceFixture) testCreateBeacons(t *testing.T) {
	// Create the first beacon
	beacon, err := c.persistence.Create(context.Background(), *c.BEACON1)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON1.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON1.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON1.Type, beacon.Type)
	assert.Equal(t, c.BEACON1.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Create the second beacon
	beacon, err = c.persistence.Create(context.Background(), *c.BEACON2)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON2.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON2.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON2.Type, beacon.Type)
	assert.Equal(t, c.BEACON2.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	// Create the third beacon
	beacon, err = c.persistence.Create(context.Background(), *c.BEACON3)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, c.BEACON3.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON3.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON3.Type, beacon.Type)
	assert.Equal(t, c.BEACON3.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)
}

func (c *BeaconsPersistenceFixture) TestCrudOperations(t *testing.T) {
	var beacon1 data1.BeaconV1

	// Create items
	c.testCreateBeacons(t)

	// Get all beacons
	page, err := c.persistence.GetPageByFilter(context.Background(),
		*cquery.NewEmptyFilterParams(), *cquery.NewEmptyPagingParams())
	assert.Nil(t, err)
	assert.NotNil(t, page)
	assert.True(t, page.HasData())
	assert.Len(t, page.Data, 3)
	beacon1 = page.Data[0].Clone()

	// Update the beacon
	beacon1.Label = "ABC"
	beacon, err := c.persistence.Update(context.Background(), beacon1)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)
	assert.Equal(t, "ABC", beacon.Label)

	// Get beacon by udi
	beacon, err = c.persistence.GetOneByUdi(context.Background(), beacon1.Udi)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Delete the beacon
	beacon, err = c.persistence.DeleteById(context.Background(), beacon1.Id)
	assert.Nil(t, err)
	assert.NotEqual(t, data1.BeaconV1{}, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	// Try to get deleted beacon
	beacon, err = c.persistence.GetOneById(context.Background(), beacon1.Id)
	assert.Nil(t, err)
	assert.Equal(t, data1.BeaconV1{}, beacon)
}

func (c *BeaconsPersistenceFixture) TestGetWithFilters(t *testing.T) {
	// Create items
	c.testCreateBeacons(t)

	filter := *cquery.NewFilterParamsFromTuples(
		"id", "1",
	)
	// Filter by id
	page, err := c.persistence.GetPageByFilter(context.Background(),
		filter,
		*cquery.NewEmptyPagingParams())
	assert.Nil(t, err)
	assert.True(t, page.HasData())
	assert.Len(t, page.Data, 1)

	// Filter by udi
	filter = *cquery.NewFilterParamsFromTuples(
		"udi", "00002",
	)
	page, err = c.persistence.GetPageByFilter(
		context.Background(),
		filter,
		*cquery.NewEmptyPagingParams())
	assert.Nil(t, err)
	assert.True(t, page.HasData())
	assert.Len(t, page.Data, 1)

	// Filter by udis
	filter = *cquery.NewFilterParamsFromTuples(
		"udis", "00001,00003",
	)
	page, err = c.persistence.GetPageByFilter(
		context.Background(),
		filter,
		*cquery.NewEmptyPagingParams())

	assert.Nil(t, err)
	assert.True(t, page.HasData())
	assert.Len(t, page.Data, 2)

	// Filter by site_id
	filter = *cquery.NewFilterParamsFromTuples(
		"site_id", "1",
	)
	page, err = c.persistence.GetPageByFilter(
		context.Background(),
		filter,
		*cquery.NewEmptyPagingParams())

	assert.Nil(t, err)
	assert.Len(t, page.Data, 2)
}

```

/test/persistence/BeaconsMemoryPersistence_test.go
```
package test_persistence

import (
	"context"
	"testing"

	persist "github.com/pip-services-samples/service-beacons-go/persistence"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
)

type BeaconsMemoryPersistenceTest struct {
	persistence *persist.BeaconsMemoryPersistence
	fixture     *BeaconsPersistenceFixture
}

func newBeaconsMemoryPersistenceTest() *BeaconsMemoryPersistenceTest {
	persistence := persist.NewBeaconsMemoryPersistence()
	persistence.Configure(context.Background(), cconf.NewEmptyConfigParams())

	fixture := NewBeaconsPersistenceFixture(persistence)

	return &BeaconsMemoryPersistenceTest{
		persistence: persistence,
		fixture:     fixture,
	}
}

func (c *BeaconsMemoryPersistenceTest) setup(t *testing.T) {
	err := c.persistence.Open(context.Background())
	if err != nil {
		t.Error("Failed to open persistence", err)
	}

	err = c.persistence.Clear(context.Background())
	if err != nil {
		t.Error("Failed to clear persistence", err)
	}
}

func (c *BeaconsMemoryPersistenceTest) teardown(t *testing.T) {
	err := c.persistence.Close(context.Background())
	if err != nil {
		t.Error("Failed to close persistence", err)
	}
}

func TestBeaconsMemoryPersistence(t *testing.T) {
	c := newBeaconsMemoryPersistenceTest()
	if c == nil {
		return
	}

	c.setup(t)
	t.Run("CRUD Operations", c.fixture.TestCrudOperations)
	c.teardown(t)

	c.setup(t)
	t.Run("Get With Filters", c.fixture.TestGetWithFilters)
	c.teardown(t)
}

```