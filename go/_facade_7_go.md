```
package test_fixture

import (
	"context"
	"time"

	bclients1 "github.com/pip-services-samples/client-beacons-gox/clients/version1"
	fbuild "github.com/pip-services-samples/pip-samples-facade-go/build"
	operations1 "github.com/pip-services-samples/pip-samples-facade-go/operations/version1"
	services1 "github.com/pip-services-samples/pip-samples-facade-go/services/version1"
	accclients1 "github.com/pip-services-users/pip-clients-accounts-go/version1"
	passclients1 "github.com/pip-services-users/pip-clients-passwords-go/version1"
	roleclients1 "github.com/pip-services-users/pip-clients-roles-go/version1"
	sessclients1 "github.com/pip-services-users/pip-clients-sessions-go/version1"
	cbuild "github.com/pip-services4/pip-services4-go/pip-services4-components-go/build"
	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	bref "github.com/pip-services4/pip-services4-go/pip-services4-container-go/refer"
	rpcbuild "github.com/pip-services4/pip-services4-go/pip-services4-http-go/build"
	httpcontr "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type TestReferences struct {
	*bref.ManagedReferences

	factory *cbuild.CompositeFactory
}

func NewTestReferences() *TestReferences {
	c := TestReferences{
		ManagedReferences: bref.NewManagedReferences(context.Background(), nil),
		factory:           cbuild.NewCompositeFactory(),
	}

	c.setupFactories()
	c.appendDependencies()
	c.configureService()
	c.createUsersAndSessions()
	return &c
}

func (c *TestReferences) setupFactories() {
	c.factory.Add(fbuild.NewClientFacadeFactory())
	c.factory.Add(fbuild.NewServiceFacadeFactory())
	c.factory.Add(rpcbuild.NewDefaultHttpFactory())
}

func (c *TestReferences) Append(descriptor *cref.Descriptor) {
	component, err := c.factory.Create(descriptor)
	if err != nil {
		return
	}
	c.Put(context.Background(), descriptor, component)
}

func (c *TestReferences) appendDependencies() {
	// Add factories
	c.Put(context.Background(), nil, c.factory)

	// Add service
	c.Put(context.Background(), nil, services1.NewFacadeServiceV1())

	// Add user management services
	c.Put(context.Background(), cref.NewDescriptor("pip-services-accounts", "client", "memory", "default", "*"), accclients1.NewAccountsMemoryClientV1(nil))
	c.Put(context.Background(), cref.NewDescriptor("pip-services-sessions", "client", "memory", "default", "*"), sessclients1.NewSessionsMemoryClientV1())
	c.Put(context.Background(), cref.NewDescriptor("pip-services-passwords", "client", "commandable-http", "default", "*"), passclients1.NewPasswordsMemoryClientV1())
	c.Put(context.Background(), cref.NewDescriptor("pip-services-roles", "client", "commandable-http", "default", "*"), roleclients1.NewRolesMemoryClientV1())

	// Add content management services
	// Beacons
	c.Put(context.Background(), cref.NewDescriptor("beacons", "client", "memory", "default", "*"), bclients1.NewBeaconsMemoryClientV1(nil))
}

func (c *TestReferences) configureService() {
	// Configure Facade service
	dependency, _ := c.GetOneRequired(cref.NewDescriptor("pip-services", "endpoint", "http", "default", "*"))
	service, ok1 := dependency.(*httpcontr.HttpEndpoint)
	if !ok1 {
		panic("TestReferences: Cant't resolv dependency 'client' to IAccountsClientV1")
	}

	service.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"root_path", "", //"/api/1.0",
		"connection.protocol", "http",
		"connection.host", "0.0.0.0",
		"connection.port", 3000,
	))

}

func (c *TestReferences) createUsersAndSessions() {
	// Create accounts
	dependency, _ := c.GetOneRequired(cref.NewDescriptor("pip-services-accounts", "client", "*", "*", "*"))
	accountsClient, ok1 := dependency.(accclients1.IAccountsClientV1)
	if !ok1 {
		panic("TestReferences: Cant't resolv dependency 'client' to IAccountsClientV1")
	}

	adminUserAccount := accclients1.AccountV1{
		Id:         TestUsers.AdminUserId,
		Login:      TestUsers.AdminUserLogin,
		Name:       TestUsers.AdminUserName,
		Active:     true,
		CreateTime: time.Time{},
	}
	accountsClient.CreateAccount("", &adminUserAccount)

	user1Account := accclients1.AccountV1{
		Id:         TestUsers.User1Id,
		Login:      TestUsers.User1Login,
		Name:       TestUsers.User1Name,
		Active:     true,
		CreateTime: time.Time{},
	}
	accountsClient.CreateAccount("", &user1Account)

	user2Account := accclients1.AccountV1{
		Id:         TestUsers.User2Id,
		Login:      TestUsers.User2Login,
		Name:       TestUsers.User2Name,
		Active:     true,
		CreateTime: time.Time{},
	}
	accountsClient.CreateAccount("", &user2Account)

	// Create opened sessions

	dependency, _ = c.GetOneRequired(cref.NewDescriptor("pip-services-sessions", "client", "*", "*", "*"))
	sessionsClient, ok2 := dependency.(sessclients1.ISessionsClientV1)
	if !ok2 {
		panic("TestReferences: Cant't resolv dependency 'client' to ISessionsClientV1")
	}

	adminUserData := c.cloneAccountToUserData(&adminUserAccount)
	adminUserData.Roles = []string{"admin"}
	session, _ := sessionsClient.OpenSession(
		"", TestUsers.AdminUserId, TestUsers.AdminUserName,
		"", "", adminUserData, nil)
	session.Id = TestUsers.AdminUserSessionId

	user1Data := c.cloneAccountToUserData(&user1Account)
	user1Data.Roles = make([]string, 0)

	session, _ = sessionsClient.OpenSession(
		"", TestUsers.User1Id, TestUsers.User1Name,
		"", "", user1Data, nil)
	session.Id = TestUsers.User1SessionId

	user2Data := c.cloneAccountToUserData(&user2Account)
	user2Data.Roles = make([]string, 0)
	session, _ = sessionsClient.OpenSession(
		"", TestUsers.User2Id, TestUsers.User2Name,
		"", "", user2Data, nil)
	session.Id = TestUsers.User2SessionId
}

func (c *TestReferences) cloneAccountToUserData(account *accclients1.AccountV1) *operations1.SessionUserV1 {
	if account == nil {
		return nil
	}
	return &operations1.SessionUserV1{
		Id:    account.Id,
		Login: account.Login,
		Name:  account.Name,

		/* Activity tracking */
		CreateTime: account.CreateTime,
		/* User preferences */
		TimeZone: account.TimeZone,
		Language: account.Language,
		Theme:    account.Theme,

		/* Custom fields */
		CustomHdr: account.CustomHdr,
		CustomDat: account.CustomDat,
	}
}

```

```
package test_fixture

import (
	"bytes"
	"encoding/json"
	"io/ioutil"
	"net/http"
	"strings"

	cerr "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/errors"
)

type TestRestClient struct {
	url string
}

func NewTestRestClient() *TestRestClient {
	c := TestRestClient{
		url: "http://localhost:3000",
	}
	return &c
}

func (c *TestRestClient) invoke(method string,
	route string, headers map[string]string, body interface{}, result interface{}) error {
	var url string = c.url + route

	method = strings.ToUpper(method)
	var bodyReader *bytes.Reader = bytes.NewReader(make([]byte, 0, 0))
	if body != nil {
		jsonBody, _ := json.Marshal(body)
		bodyReader = bytes.NewReader(jsonBody)
	}

	req, err := http.NewRequest(method, url, bodyReader)

	if err != nil {
		return err
	}
	// Set headers
	req.Header.Set("Accept", "application/json")
	if headers != nil && len(headers) > 0 {
		for k, v := range headers {
			req.Header.Set(k, v)
		}
	}
	client := http.Client{}
	response, respErr := client.Do(req)

	if respErr != nil {
		return respErr
	}

	if response.StatusCode == 204 {
		return nil
	}

	resBody, bodyErr := ioutil.ReadAll(response.Body)
	if bodyErr != nil {
		return bodyErr
	}

	if response.StatusCode >= 400 {
		appErr := cerr.ApplicationError{}
		json.Unmarshal(resBody, &appErr)
		return &appErr
	}

	if result == nil {
		return nil
	}

	jsonErr := json.Unmarshal(resBody, result)
	return jsonErr
}

func (c *TestRestClient) Get(path string, result interface{}) error {
	return c.invoke("get", path, nil, nil, result)
}

func (c *TestRestClient) Head(path string, result interface{}) error {
	return c.invoke("head", path, nil, nil, result)
}

func (c *TestRestClient) Post(path string, params interface{}, result interface{}) error {
	return c.invoke("post", path, nil, params, result)
}

func (c *TestRestClient) Put(path string, params interface{}, result interface{}) error {
	return c.invoke("put", path, nil, params, result)
}

func (c *TestRestClient) Del(path string, result interface{}) error {
	return c.invoke("delete", path, nil, nil, result)
}

func (c *TestRestClient) GetAsUser(sessionId string, path string, result interface{}) error {
	return c.invoke("get", path, map[string]string{"x-session-id": sessionId}, nil, result)
}

func (c *TestRestClient) HeadAsUser(sessionId string, path string, result interface{}) error {
	return c.invoke("head", path, map[string]string{"x-session-id": sessionId}, nil, result)
}

func (c *TestRestClient) PostAsUser(sessionId string, path string, params interface{}, result interface{}) error {
	return c.invoke("post", path, map[string]string{"x-session-id": sessionId}, params, result)
}

func (c *TestRestClient) PutAsUser(sessionId string, path string, params interface{}, result interface{}) error {
	return c.invoke("put", path, map[string]string{"x-session-id": sessionId}, params, result)
}

func (c *TestRestClient) DelAsUser(sessionId string, path string, result interface{}) error {
	return c.invoke("delete", path, map[string]string{"x-session-id": sessionId}, nil, result)
}

```

```
package test_fixture

var TestUsers tTestUsers = NewTTestUsers()

type tTestUsers struct {
	AdminUserId        string
	AdminUserName      string
	AdminUserLogin     string
	AdminUserPassword  string
	AdminUserSessionId string

	User1Id        string
	User1Name      string
	User1Login     string
	User1Password  string
	User1SessionId string

	User2Id        string
	User2Name      string
	User2Login     string
	User2Password  string
	User2SessionId string
}

func NewTTestUsers() tTestUsers {
	c := tTestUsers{
		AdminUserId:        "1",
		AdminUserName:      "Admin User",
		AdminUserLogin:     "admin",
		AdminUserPassword:  "pwd123",
		AdminUserSessionId: "111",

		User1Id:        "2",
		User1Name:      "User #1",
		User1Login:     "user1",
		User1Password:  "pwd123",
		User1SessionId: "222",

		User2Id:        "3",
		User2Name:      "User #2",
		User2Login:     "user2",
		User2Password:  "pwd123",
		User2SessionId: "333",
	}
	return c
}

```

```
package test_operations

import (
	"context"
	"testing"

	testfixture "github.com/pip-services-samples/pip-samples-facade-go/test/fixture"
	data1 "github.com/pip-services-samples/service-beacons-gox/data/version1"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	"github.com/stretchr/testify/assert"
)

type beaconsRestServiceV1Test struct {
	BEACON1    *data1.BeaconV1
	BEACON2    *data1.BeaconV1
	references *testfixture.TestReferences
	rest       *testfixture.TestRestClient
}

func newBeaconsRestServiceV1Test() *beaconsRestServiceV1Test {
	BEACON1 := &data1.BeaconV1{
		Id:     "1",
		Udi:    "00001",
		Type:   data1.AltBeacon,
		SiteId: "1",
		Label:  "TestBeacon1",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: [][]float32{{0.0, 0.0}}},
		Radius: 50,
	}

	BEACON2 := &data1.BeaconV1{
		Id:     "2",
		Udi:    "00002",
		Type:   data1.IBeacon,
		SiteId: "1",
		Label:  "TestBeacon2",
		Center: data1.GeoPointV1{Type: "Point", Coordinates: [][]float32{{2.0, 2.0}}},
		Radius: 70,
	}

	return &beaconsRestServiceV1Test{
		BEACON1: BEACON1,
		BEACON2: BEACON2,
	}
}

func (c *beaconsRestServiceV1Test) setup(t *testing.T) {
	c.rest = testfixture.NewTestRestClient()
	c.references = testfixture.NewTestReferences()
	err := c.references.Open(context.Background())
	if err != nil {
		t.Error("Failed to open references", err)
	}
}

func (c *beaconsRestServiceV1Test) teardown(t *testing.T) {
	c.rest = nil
	err := c.references.Close(context.Background())
	if err != nil {
		t.Error("Failed to close references", err)
	}
}

func (c *beaconsRestServiceV1Test) testCrudOperations(t *testing.T) {
	var beacon1 *data1.BeaconV1

	var beacon data1.BeaconV1

	err := c.rest.PostAsUser(testfixture.TestUsers.AdminUserSessionId, "/api/v1/beacons", c.BEACON1, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, c.BEACON1.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON1.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON1.Type, beacon.Type)
	assert.Equal(t, c.BEACON1.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	err = c.rest.PostAsUser(testfixture.TestUsers.AdminUserSessionId, "/api/v1/beacons", c.BEACON2, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, c.BEACON2.Udi, beacon.Udi)
	assert.Equal(t, c.BEACON2.SiteId, beacon.SiteId)
	assert.Equal(t, c.BEACON2.Type, beacon.Type)
	assert.Equal(t, c.BEACON2.Label, beacon.Label)
	assert.NotNil(t, beacon.Center)

	var page data1.BeaconV1DataPage
	err = c.rest.GetAsUser(testfixture.TestUsers.AdminUserSessionId, "/api/v1/beacons", &page)
	assert.Nil(t, err)
	assert.NotNil(t, page)
	assert.Len(t, page.Data, 2)
	beacon1 = page.Data[0]

	// Update the beacon
	beacon1.Label = "ABC"
	err = c.rest.PutAsUser(testfixture.TestUsers.AdminUserSessionId, "/api/v1/beacons", beacon1, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)
	assert.Equal(t, "ABC", beacon.Label)

	err = c.rest.GetAsUser(testfixture.TestUsers.User1SessionId, "/api/v1/beacons/udi/"+beacon1.Udi+"?user_id="+testfixture.TestUsers.User1Id, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	//Calculate position for one beacon
	body := cdata.NewAnyValueMapFromTuples(
		"site_id", "1",
		"udis", []string{"00001"},
	)
	var position data1.GeoPointV1
	err = c.rest.PostAsUser(testfixture.TestUsers.User1SessionId, "/api/v1/beacons/position", body.Value(), &position)
	assert.Nil(t, err)
	assert.NotNil(t, position)
	assert.Equal(t, "Point", position.Type)
	assert.Equal(t, (float32)(0.0), position.Coordinates[0][0])
	assert.Equal(t, (float32)(0.0), position.Coordinates[0][1])

	err = c.rest.DelAsUser(testfixture.TestUsers.AdminUserSessionId, "/api/v1/beacons/"+beacon1.Id, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Equal(t, beacon1.Id, beacon.Id)

	beacon = data1.BeaconV1{}
	err = c.rest.GetAsUser(testfixture.TestUsers.User1SessionId, "/api/v1/beacons/"+beacon1.Id+"?user_id="+testfixture.TestUsers.User1Id, &beacon)
	assert.Nil(t, err)
	assert.NotNil(t, beacon)
	assert.Empty(t, beacon)
}

func TestBeaconsRestServiceV1(t *testing.T) {
	c := newBeaconsRestServiceV1Test()

	c.setup(t)
	t.Run("CRUD Operations", c.testCrudOperations)
	c.teardown(t)

}

```

```
package test_operations

import (
	"context"
	"testing"

	testfixture "github.com/pip-services-samples/pip-samples-facade-go/test/fixture"
	"github.com/stretchr/testify/assert"
)

type sessionRoutesV1Test struct {
	references *testfixture.TestReferences
	rest       *testfixture.TestRestClient
	user       map[string]string
}

func newSessionRoutesV1Test() *sessionRoutesV1Test {
	c := &sessionRoutesV1Test{
		user: make(map[string]string, 0),
	}

	c.user["login"] = "test"
	c.user["name"] = "Test User"
	c.user["email"] = "test@conceptual.vision"
	c.user["password"] = "test123"

	return c
}

func (c *sessionRoutesV1Test) setup(t *testing.T) {
	c.rest = testfixture.NewTestRestClient()
	c.references = testfixture.NewTestReferences()
	err := c.references.Open(context.Background())
	if err != nil {
		t.Error("Failed to open references", err)
	}
}

func (c *sessionRoutesV1Test) teardown(t *testing.T) {
	c.rest = nil
	err := c.references.Close(context.Background())
	if err != nil {
		t.Error("Failed to close references", err)
	}
}

func (c *sessionRoutesV1Test) testSignupNewUser(t *testing.T) {

	session := make(map[string]interface{})
	err := c.rest.Post("/api/v1/users/signup", c.user, &session)

	assert.Nil(t, err)
	assert.NotNil(t, session)
	assert.NotNil(t, session["id"])
	assert.Equal(t, session["user_name"], c.user["name"])
}

func (c *sessionRoutesV1Test) testNotSignupWithTheSameEmail(t *testing.T) {

	// Sign up
	session := make(map[string]interface{})
	err := c.rest.Post("/api/v1/users/signup", c.user, &session)
	assert.Nil(t, err)
	// Try to sign up again
	err = c.rest.Post("/api/v1/users/signup", c.user, &session)
	assert.NotNil(t, err)
}

func (c *sessionRoutesV1Test) testShouldSignout(t *testing.T) {
	result := make(map[string]interface{})
	err := c.rest.Post("/api/v1/users/signout", nil, &result)
	assert.Nil(t, err)
}

func (c *sessionRoutesV1Test) testShouldSigninWithEmailAndPassword(t *testing.T) {

	// Sign up
	session := make(map[string]interface{})
	err := c.rest.Post("/api/v1/users/signup",
		c.user, &session)
	assert.Nil(t, err)

	// Sign in with username
	err = c.rest.Post("/api/v1/users/signin",
		map[string]string{
			"login":    c.user["login"],
			"password": c.user["password"],
		}, &session)
	assert.Nil(t, err)
}

func TestSignupNewUser(t *testing.T) {
	c := newSessionRoutesV1Test()

	c.setup(t)
	t.Run("Signup New User", c.testSignupNewUser)
	c.teardown(t)
}

func TestNotSignupWithTheSameEmail(t *testing.T) {
	c := newSessionRoutesV1Test()
	c.setup(t)
	t.Run("Not Signup With The Same Email", c.testNotSignupWithTheSameEmail)
	c.teardown(t)
}
func TestShouldSignout(t *testing.T) {
	c := newSessionRoutesV1Test()
	c.setup(t)
	t.Run("Should Signout", c.testShouldSignout)
	c.teardown(t)

}
func TestShouldSigninWithEmailAndPassword(t *testing.T) {
	c := newSessionRoutesV1Test()
	c.setup(t)
	t.Run("Should Signin With Email And Password", c.testShouldSigninWithEmailAndPassword)
	c.teardown(t)
}

```

```
go test ./...

```
