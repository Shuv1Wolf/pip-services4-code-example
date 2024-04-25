```
/bin
/config
/docker

/build
/container
/data
└───/version1
/service
/persistence
/controllers
└───/version1

/test
└───/service
└───/persistence
└───/controllers
    └───/version1
```

**/go.mod**

```
module github.com/pip-services-samples/service-beacons-go

go 1.18

require (
	github.com/pip-services4/pip-services4-go/pip-services4-commons-go v0.0.1-2
	github.com/pip-services4/pip-services4-go/pip-services4-components-go v0.0.1-2
	github.com/pip-services4/pip-services4-go/pip-services4-container-go v0.0.0-20231031045919-47eb45838aeb
	github.com/pip-services4/pip-services4-go/pip-services4-data-go v0.0.1-2
	github.com/pip-services4/pip-services4-go/pip-services4-http-go v0.0.0-20231031045919-47eb45838aeb
	github.com/pip-services4/pip-services4-go/pip-services4-mongodb-go v0.0.1-3
	github.com/pip-services4/pip-services4-go/pip-services4-persistence-go v0.0.0-20230718211601-c5e741d55d0e
	github.com/pip-services4/pip-services4-go/pip-services4-postgres-go v0.0.1-2
	github.com/pip-services4/pip-services4-go/pip-services4-rpc-go v0.0.0-20231031045919-47eb45838aeb
	github.com/stretchr/testify v1.9.0
	go.mongodb.org/mongo-driver v1.12.0
)

require (
	github.com/davecgh/go-spew v1.1.1 // indirect
	github.com/golang/snappy v0.0.1 // indirect
	github.com/google/uuid v1.6.0 // indirect
	github.com/jackc/chunkreader/v2 v2.0.1 // indirect
	github.com/jackc/pgconn v1.14.0 // indirect
	github.com/jackc/pgio v1.0.0 // indirect
	github.com/jackc/pgpassfile v1.0.0 // indirect
	github.com/jackc/pgproto3/v2 v2.3.2 // indirect
	github.com/jackc/pgservicefile v0.0.0-20221227161230-091c0ba34f0a // indirect
	github.com/jackc/pgtype v1.14.0 // indirect
	github.com/jackc/pgx/v4 v4.18.1 // indirect
	github.com/jackc/puddle v1.3.0 // indirect
	github.com/jinzhu/copier v0.3.5 // indirect
	github.com/klauspost/compress v1.13.6 // indirect
	github.com/montanaflynn/stats v0.0.0-20171201202039-1bf9dbcd8cbe // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-config-go v0.0.0-20230718211601-c5e741d55d0e // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-expressions-go v0.0.0-20230621165553-e2896e10dc3d // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-logic-go v0.0.0-20231031045919-47eb45838aeb // indirect
	github.com/pip-services4/pip-services4-go/pip-services4-observability-go v0.0.0-20230718211601-c5e741d55d0e // indirect
	github.com/pmezard/go-difflib v1.0.0 // indirect
	github.com/rs/cors v1.9.0 // indirect
	github.com/xdg-go/pbkdf2 v1.0.0 // indirect
	github.com/xdg-go/scram v1.1.2 // indirect
	github.com/xdg-go/stringprep v1.0.4 // indirect
	github.com/youmark/pkcs8 v0.0.0-20181117223130-1be2e3e5546d // indirect
	goji.io v2.0.2+incompatible // indirect
	golang.org/x/crypto v0.6.0 // indirect
	golang.org/x/sync v0.0.0-20220722155255-886fb9371eb4 // indirect
	golang.org/x/text v0.7.0 // indirect
	gopkg.in/yaml.v2 v2.4.0 // indirect
	gopkg.in/yaml.v3 v3.0.1 // indirect
)

```

```
go mod download
```
