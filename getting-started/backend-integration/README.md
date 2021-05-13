# Integrate with your backend

There are two ways to authenticate requests to your application.

**Forward authentication with reserve proxy**

Setup reserve proxy that supports forward authentication \(e.g. NGINX, Traefik... etc\) in your infrastructure. Your reserve proxy forwards the incoming HTTP request without the request body to the Authgear resolver endpoint. Authgear resolver response HTTP headers, reserve proxy update the request with those extra headers which contain user authentication information. Your backend can read those headers to determine whether the incoming HTTP request is authenticated or not

**Validate JSON Web Token \(JWT\) in your application server**

With this option turn on in your application, Authgear will issue JWT as the access token. Without setting the reverse proxy, your backend can use your Authgear JWKS to verify the JWT and decode user information from the token.

## Comparison

|  | **Forward authentication with reserve proxy** | **Validate JSON Web Token \(JWT\) in your application server** |
| :--- | :--- | :--- |
| Integration difficulties | ☹Medium, need setup extra reserve proxy to resolve authentication information | 😀Easy, you only need to add code in your application to validate and decode JWT |
| Reliability | 😀Update near real-time, based on your reserve proxy cache setting | ☹JWT only updates when expire, that means before the token expiry, your application may see the user is valid even s/he has been disabled |

## Setup guides

**Forward authentication with reserve proxy**

{% page-ref page="nginx.md" %}

**Validate JSON Web Token \(JWT\) in your application server**

{% page-ref page="jwt.md" %}



\*\*\*\*

\*\*\*\*



