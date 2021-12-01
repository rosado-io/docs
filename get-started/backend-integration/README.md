---
description: >-
  Decide how your backend application server authenticate the incoming HTTP
  requests.
---

# Backend Integration

For Mobile App or Single Page Web App or Website, each request from the client to your application server should contain an access token or a cookie. Your backend server should validate them for each HTTP request.

There are two approaches to verify the requests, validate JWT in the your server or forward to Authgear Resolver Endpoint.

**Validate JSON Web Token (JWT) in your application server**

This approach is only available for **Token-based authentication**.

With the **Issue JWT as access token** option turned on in your application, Authgear will issue JWT as access tokens. The incoming HTTP requests should include the access token in their `Authorization` headers. Without setting the reverse proxy, your backend server can use your Authgear JWKS to verify the request and decode user information from the JWT access token.

**Forward Authentication to Authgear Resolver Endpoint**

This approach is available for both **Token-based** and **Cookie-based authentication**.

The recommended but more complicated approach is to forward each incoming HTTP request to the Authgear Resolver Endpoint to verify the access token or cookie.&#x20;

You can forward the requests without the request body to the resolver endpoint. Authgear will look at the `Authorization` and `Cookie` in the HTTP header, verify the token, and respond HTTP 200 with `X-Authgear-` headers for session validity, the user id...etc.

If you use a popular reverse proxy on your deployment, such as NGINX, Traefik, etc, you can configure it with a few simple lines of forward auth config. Your backend should read the returned headers to determine the identity of the user of the HTTP request.

## Comparison

|                          | **Validate JSON Web Token (JWT) in your application server**                                                                                                               | Forward Authentication to Authgear Resolver Endpoint                                                      |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Reliability              | <p><strong>Medium</strong><br>JWT only updates when expire. That means before the token expiry, your application may see the user is valid even they has been disabled</p> | <p><strong>High</strong><br>Update near real-time, based on your reserve proxy cache setting</p>          |
| Integration difficulties | <p><strong>Easy</strong><br>You only need to add code in your application to validate and decode JWT</p>                                                                   | <p><strong>Medium</strong><br>Need to setup extra reverse proxy to resolve authentication information</p> |

## Setup guides

**Validate JSON Web Token (JWT) in your application server**

{% content-ref url="jwt.md" %}
[jwt.md](jwt.md)
{% endcontent-ref %}

**Forward authentication with Authgear Resolver Endpoint**

{% content-ref url="nginx.md" %}
[nginx.md](nginx.md)
{% endcontent-ref %}
