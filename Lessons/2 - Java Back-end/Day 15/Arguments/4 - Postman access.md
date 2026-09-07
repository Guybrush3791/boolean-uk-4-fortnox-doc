# Postman access

## Learning Objectives

- Verify the public and protected endpoint behaviour before authentication.
- Obtain a Keycloak access token through Postman's OAuth2 helper.
- Send the token in the request header and observe the protected response.

## Verify both boundaries without a token

Create a `GET` request for `http://localhost:4000/public/test`. On the **Authorization** tab, select **No Auth**, then send the request. It should return `200 OK` with:

```text file:"Public response"
Public Hello, World!
```

Now create a `GET` request for `http://localhost:4000/test` and keep **No Auth**. It should return `401 Unauthorized`. This proves that the filter chain rejects the request before the protected controller method runs.

## Get an OAuth2 access token

On the protected request, open **Authorization**, select **OAuth 2.0**, and configure a new token:

| Key | Value |
| :--- | :--- |
| Add authorization data to | **Request Headers** |
| Token name | `springboot-realm-1` |
| Grant type | **Authorization Code (PKCE)** |
| Callback URL | `https://oauth.pstmn.io/v1/callback` |
| Auth URL | `http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/auth` |
| Access token URL | `http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/token` |
| Client ID | `springboot-realm-1` |
| Scope | `openid` |

Enter the values as shown in the image.

![[Postman OAuth2 Setup|1200]]

Select **Get New Access Token** and log in when Keycloak prompts.

![[Postman OAuth2 Login Route|1600]]

When the process finishes, select **Use Token**. The top of the OAuth2 Authorization tab shows the token in use, its expiration date and the **Refresh** button.

## Access the protected endpoint

Send `GET http://localhost:4000/test` again. Postman now adds the access token as a bearer value in the `Authorization` request header. The API should return `200 OK` with:

```text file:"Protected response"
Private Hello, World
```

The controller code did not change between the failed and successful requests. The difference is the token checked by Spring Security before the request reaches `getPrivateHello()`.

## Compare the observable results

| Request | Token | Expected result |
| --- | --- | --- |
| `GET /public/test` | None | `200 OK` and `Public Hello, World!` |
| `GET /test` | None | `401 Unauthorized` |
| `GET /test` | Valid Keycloak access token | `200 OK` and `Private Hello, World` |
| `GET /test` | Invalid, expired or wrong-issuer token | `401 Unauthorized` |

Use **No Auth** for `/public/**`. Use an OAuth2 bearer token for every other route. A missing, expired or invalid token should produce `401 Unauthorized`; a valid token allows the request to continue to the controller.

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__blocks/Links]]
