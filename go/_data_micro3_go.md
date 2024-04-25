/data/version1/BeaconV1.go
```
package data1

type BeaconV1 struct {
	Id     string     `json:"id" bson:"_id"`
	SiteId string     `json:"site_id" bson:"site_id"`
	Type   string     `json:"type" bson:"type"`
	Udi    string     `json:"udi" bson:"udi"`
	Label  string     `json:"label" bson:"label"`
	Center GeoPointV1 `json:"center" bson:"center"` // GeoJson
	Radius float32    `json:"radius" bson:"radius"`
}

func (b BeaconV1) Clone() BeaconV1 {
	return BeaconV1{
		Id:     b.Id,
		SiteId: b.SiteId,
		Type:   b.Type,
		Udi:    b.Udi,
		Label:  b.Label,
		Center: b.Center.Clone(),
		Radius: b.Radius,
	}
}

```

/data/version1/BeaconTypeV1.go
```
package data1

const Unknown = "unknown"
const AltBeacon = "altbeacon"
const IBeacon = "ibeacon"
const EddyStoneUdi = "eddystone-udi"

```

/data/version1/BeaconV1Schema.go
```
package data1

import (
	cconv "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/convert"
	cvalid "github.com/pip-services4/pip-services4-go/pip-services4-data-go/validate"
)

type BeaconV1Schema struct {
	cvalid.ObjectSchema
}

func NewBeaconV1Schema() *BeaconV1Schema {
	c := BeaconV1Schema{}
	c.ObjectSchema = *cvalid.NewObjectSchema()

	c.WithOptionalProperty("id", cconv.String)
	c.WithRequiredProperty("site_id", cconv.String)
	c.WithOptionalProperty("type", cconv.String)
	c.WithRequiredProperty("udi", cconv.String)
	c.WithOptionalProperty("label", cconv.String)
	c.WithOptionalProperty("center", cconv.Map)
	c.WithOptionalProperty("radius", cconv.Double)
	return &c
}

```

