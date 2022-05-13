# API for Client Applications (OIDC 2.0)

If your app is `myapp`, the endpoint of your app is `https://myapp.authgearapps.com`.

Unless otherwise specified, all paths mentioned here are relative to the endpoint of your app.

## /.well-known/openid-configuration

This endpoint serves a JSON document containing the OpenID Connect configuration of your app. That includes the authorization endpoint, the token endpoint and the JWKs endpoint.

Here is [an example of how it looks](https://accounts.portal.authgearapps.com/.well-known/openid-configuration).

## /.well-known/oauth-authorization-server

This endpoint serves a JSON document containing the authorization server metadata of your app. That includes the authorization endpoint, the token endpoint and the JWKs endpoint.

Here is [an example of how it looks](https://accounts.portal.authgearapps.com/.well-known/oauth-authorization-server).

## /\_resolver/resolve

The endpoint serves as a resolver to check the access token or cookie in the request headers. Forward incoming HTTP requests to this endpoint and the resolver will adds the `x-authgear-` headers the to response.

See the list of `x-authgear-` headers in the specs [here](https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md).

See implementation examples [here](../get-started/backend-integration/nginx.md).

## /

This endpoint is the entrypoint of the Web UI. You can visit it if you want to try your configuration. However, this is NOT the authorization endpoint. You must use our SDK to initiate the authentication.

## /settings

User settings UI

## /\_api/admin/graphql

Admin GraphQL API endpoint. For usage details, please check [Admin APIs](admin-apis/).

## /oauth2/userinfo

The UserInfo Endpoint is an OAuth 2.0 Protected Resource that returns Claims about the authenticated End-User. When the client presents with a valid Access Token, the endpoint responds with the claims packaged in a JSON object. The claims are also the attributes of the [User Profile](../integrate/user-profile.md).&#x20;
