# API for Client Applications \(OIDC 2.0\)

If your app is `myapp`, the endpoint of your app is [https://myapp.authgearapps.com](https://myapp.authgearapps.com).

Unless otherwise specified, all paths mentioned here are relative to the endpoint of your app.

### /.well-known/openid-configuration

This endpoint serves a JSON document containing the OpenID Connect configuration of your app.
That includes the authorization endpoint, the token endpoint and the JWKs endpoint.

Here is [an example of how it looks](https://accounts.portal.authgearapps.com/.well-known/openid-configuration).

### /

This endpoint is the entrypoint of the Web UI.
You can visit it if you want to try your configuration.
However, this is NOT the authorization endpoint.
You must use our SDK to initiate the authentication.
