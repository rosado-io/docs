# Backend Integration

For Mobile App or Single Page Web App or Website, each request from the client to your application server should contain the access tokens or cookie, which your backend should validate for each HTTP requests.

**Forward Authentication to Authgear Resolver Endpoint** 

The recommended but more complicated approach is to forward each incoming HTTP request to the Authgear resolver endpoint to verify the access token or cookie. You can forward the requests without the request body to the endpoint, Authgear will look at the `Authorization` and `Cookie` in the HTTP header, verify the token, and respond HTTP 200 with `X-Authgear-` headers for session validity, the user id...etc. 

If you use a popular reverse proxy on your deployment, such as NGINX, Traefik, etc, you can configure it with a few simple lines of forward auth config. Your backend should read the headers returned to determine who is the user in the HTTP request.

**Validate JSON Web Token \(JWT\) in your application server**

With this option turn on in your application, Authgear will issue JWT as the access token. Without setting the reverse proxy, your backend can use your Authgear JWKS to verify the JWT and decode user information from the token.

## Comparison

|  | Forward Authentication to Authgear Resolver Endpoint | **Validate JSON Web Token \(JWT\) in your application server** |
| :--- | :--- | :--- |
| Integration difficulties | ⛔️ Medium, need setup extra reserve proxy to resolve authentication information | ✅ Easy, you only need to add code in your application to validate and decode JWT |
| Reliability | ✅ Update near real-time, based on your reserve proxy cache setting | ⛔️ JWT only updates when expire, that means before the token expiry, your application may see the user is valid even s/he has been disabled |

## Setup guides

**Forward authentication with reserve proxy**

{% page-ref page="nginx.md" %}

**Validate JSON Web Token \(JWT\) in your application server**

{% page-ref page="jwt.md" %}



