# SSO Login and Identity Provider

## Learning Objectives

- Explain why a protected endpoint needs an authenticated caller.
- Identify the user, OAuth2 client, identity provider and resource server in this lesson.
- Trace a bearer access token from Keycloak to a protected controller endpoint.

## The limit of an unprotected API

Without a security rule, a mapped controller endpoint responds without first proving who made the request. The `JavaOidc` project makes that boundary visible with only two controller methods:

```text file:"Test controller routes"
GET /public/test -> public response
GET /test        -> protected response
```

Authentication answers **who is making the request**. Authorisation answers **what that authenticated caller may access**. This lesson places both decisions before the controller:

```text file:"Secured request route"
request
    -> Spring SecurityFilterChain
    -> AuthTestController, when access is allowed
```

## SSO and the identity provider

**Single Sign-On (SSO)** is an authentication pattern in which a user signs in once with a trusted **identity provider (IdP)** and can then obtain access to connected applications without giving each application a password.

Keycloak is the IdP in this lesson. It maintains users, checks credentials and issues signed access tokens. The Spring Boot API never receives the user's Keycloak password.

The access token is a **JSON Web Token (JWT)**. A JWT is a signed value containing claims about its issuer, subject, expiry time and granted access. A REST client sends it in the HTTP header:

```http file:"Bearer token request header"
Authorization: Bearer <access-token>
```

The API validates an **access token**, which grants access to a resource. An **ID token** tells an OAuth2 client about the authenticated user. Postman may receive both during an OpenID Connect flow, but `JavaOidc` uses the access token for the protected API request.

## The four roles in this lesson

| Role | Concrete part of the lesson | Responsibility |
| --- | --- | --- |
| User | The Keycloak user created in the dashboard | Enters credentials at Keycloak |
| OAuth2 client | Postman | Starts the login flow and receives an access token |
| Identity provider | Keycloak on `http://localhost:8080` | Authenticates the user and signs the token |
| Resource server | `JavaOidc` on `http://localhost:4000` | Validates the bearer token before running protected code |

The resource server does not show a login page or create a browser session. Postman opens the Keycloak login flow, then sends the resulting token to the API:

```text file:"OIDC resource server flow"
user
    -> Postman
    -> Keycloak login
    -> access token returned to Postman
    -> GET /test with Authorization: Bearer <token>
    -> Spring Security validates the JWT
    -> AuthTestController
```

`GET /public/test` will be the deliberate exception. It reaches the controller without a token. `GET /test` requires a valid Keycloak access token.

Use an IdP when several applications should trust one authentication system. Keep the API stateless by validating a bearer token on each protected request instead of storing a login session in the API.

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__blocks/Links]]
