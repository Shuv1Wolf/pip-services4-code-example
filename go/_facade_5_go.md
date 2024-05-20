```
package controllers1

import (
	"net/http"

	"github.com/gorilla/mux"
	cdata "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/data"
	cerr "github.com/pip-services4/pip-services4-go/pip-services4-commons-go/errors"
	httpauth "github.com/pip-services4/pip-services4-go/pip-services4-http-go/auth"
	httpcontr "github.com/pip-services4/pip-services4-go/pip-services4-http-go/controllers"
)

type AuthorizerV1 struct {
	basicAuth httpauth.BasicAuthorizer
	roleAuth  httpauth.RoleAuthorizer
}

func NewAuthorizerV1() *AuthorizerV1 {
	c := &AuthorizerV1{
		basicAuth: httpauth.BasicAuthorizer{},
		roleAuth:  httpauth.RoleAuthorizer{},
	}
	return c
}

// Anybody who entered the system
func (c *AuthorizerV1) Anybody() func(res http.ResponseWriter, req *http.Request, next http.HandlerFunc) {
	return c.basicAuth.Anybody()
}

// Only registered and authenticated users
func (c *AuthorizerV1) Signed() func(res http.ResponseWriter, req *http.Request, next http.HandlerFunc) {
	return c.basicAuth.Signed()
}

// System administrator
func (c *AuthorizerV1) Admin() func(res http.ResponseWriter, req *http.Request, next http.HandlerFunc) {
	return c.roleAuth.UserInRole("admin")
}

// Only the user session owner
func (c *AuthorizerV1) Owner(idParam string) func(res http.ResponseWriter, req *http.Request, next http.HandlerFunc) {
	if idParam == "" {
		idParam = "user_id"
	}
	return func(res http.ResponseWriter, req *http.Request, next http.HandlerFunc) {

		user, ok := req.Context().Value("user").(cdata.AnyValueMap)
		partyId := req.URL.Query().Get(idParam)
		if partyId == "" {
			partyId = mux.Vars(req)[idParam]
		}

		if !ok {
			httpcontr.HttpResponseSender.SendError(
				res, req,
				cerr.NewUnauthorizedError(
					"", "NOT_SIGNED",
					"User must be signed in to perform c operation").WithStatus(401),
			)
		} else if partyId == "" {
			httpcontr.HttpResponseSender.SendError(
				res, req,
				cerr.NewUnauthorizedError(
					"", "NO_USER_ID",
					"User id is not defined").WithStatus(401),
			)
		} else {
			isOwner := partyId == user.GetAsString("id")

			if !isOwner {
				httpcontr.HttpResponseSender.SendError(
					res, req,
					cerr.NewUnauthorizedError(
						"", "NOT_OWNER", "Only user owner access is allowed").WithDetails("user_id", partyId).WithStatus(403),
				)
			} else {
				next.ServeHTTP(res, req)
			}
		}
	}
}

```
