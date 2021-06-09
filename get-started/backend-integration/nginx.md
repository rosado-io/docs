---
description: >-
  Authenticate the incoming HTTP requests by forwarding Authentication to
  Authgear Resolver Endpoint
---

# Forward Authentication to Authgear Resolver Endpoint

In this section, we will explain how to set up reverse proxy in NGINX to protect your app server from unauthorized access with the Authgear resolver.

You can also use the forward authentication features of the other popular reverse proxy. e.g.

* [Traefik ForwardAuth middleware](https://doc.traefik.io/traefik/middlewares/forwardauth/)

## Authgear Resolver Endpoint

Authgear provides an endpoint for forward authentication. Subrequests should be made to the following endpoint for authentication.

`https://<your_app_endpoint>/_resolver/resolve`

## How Forward Authentication works

![](../../.gitbook/assets/how-forward-authentication-works.png)

1. After the user is logged in, send application request to your server from the client app with access token/cookies.
2. Set up a reverse proxy in your infrastructure to authenticate HTTP requests. The reverse proxy will forward the incoming HTTP requests without the request body to the Authgear Resolver Endpoint.
3. Authgear resolver parses the access token and returns HTTP headers including the user login state. The headers are starting with `x-authgear-`.
4. You have to instruct your reverse proxy to include those extra headers, before forwarding the request to your backend server.
5. Your backend server looks at the headers and response to the client app accordingly. e.g. Returns user's content or HTTP 401 if the user is not logged in.

There are so many reverse proxies available in the wild. So here we are going to illustrate the idea of using Nginx as the reverse proxy.

## Using Nginx as the reverse proxy

We will use the module `auth_request` in NGINX. The module is not built by default, it should be enabled with the `--with-http_auth_request_module`configuration parameter.

Run this command and verify that the output includes `--with-http_auth_request_module`:

```bash
$ nginx -V 2>&1 | grep -- 'http_auth_request_module'
```

The trick here is to declare an internal `location` and use `auth_request` to initiate a subrequest to the resolved endpoint.

### Example configuration

```text
server {
  # Use variable in proxy_pass with resolver to respect DNS TTL.
  # Note that /etc/hosts and /etc/resolv.conf are NOT consulted if resolver is used.
  # See https://www.nginx.com/blog/dns-service-discovery-nginx-plus/
  resolver 8.8.8.8;
  
  # Location that requires request authentication
  location / {
    set $backend http://www.mycompany.com;
    proxy_pass $backend;
    proxy_set_header Host $host;
    # Specify the auth_request directive to initiate subrequests to the to the internal location.
    # This corresponds to the Step 2.
    auth_request /_auth;

    # Copy the `x-authgear-*` headers from the response of the subrequest to Nginx variables.
    # This corresponds to the Step 3.
    auth_request_set $x_authgear_session_valid $upstream_http_x_authgear_session_valid;
    auth_request_set $x_authgear_user_id $upstream_http_x_authgear_user_id;
    auth_request_set $x_authgear_user_anonymous $upstream_http_x_authgear_user_anonymous;
    auth_request_set $x_authgear_user_verified $upstream_http_x_authgear_user_verified;
    auth_request_set $x_authgear_session_acr $upstream_http_x_authgear_session_acr;
    auth_request_set $x_authgear_session_amr $upstream_http_x_authgear_session_amr;

    # Include the headers in the request that will be sent to your backend server.
    # This corresponds to the Step 4.
    proxy_set_header x-authgear-session-valid $x_authgear_session_valid;
    proxy_set_header x-authgear-user-id $x_authgear_user_id;
    proxy_set_header x-authgear-user-anonymous $x_authgear_user_anonymous;
    proxy_set_header x-authgear-user-verified $x_authgear_user_verified;
    proxy_set_header x-authgear-session-acr $x_authgear_session_acr;
    proxy_set_header x-authgear-session-amr $x_authgear_session_amr;

    # Your backend must inspect the request headers to determine whether the request is authenticated or not.
    # This corresponds to the Step 5.
  }

  location = /_auth {
    # Set this location for internal use only
    internal;
    set $resolver https://<YOUR_AUTHGEAR_ENDPOINT>/_resolver/resolve;
    proxy_pass $resolver;
    # The body is supposed to be consumed by your backend server.
    # Pass only the headers to the resolver
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
  }
}
```

{% hint style="info" %}
See docs for `ngx_http_auth_request_module` in NGINX for more details. [http://nginx.org/en/docs/http/ngx\_http\_auth\_request\_module.html](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html)
{% endhint %}

## Optimizing the performance

If the reverse proxy, Authgear, and your backend server are in different regions, authenticating every request could result in a huge downgrade in the performance.

You may consider enabling caching.

```text
http {
  # ...
  proxy_cache_path /tmp/cache keys_zone=auth_cache:10m;

  # The server block.
  server {
    # ...
    location = /_auth {
      # ...
      proxy_cache auth_cache;
      proxy_cache_key "$cookie_session|$http_authorization";
      proxy_cache_valid 200 10m;  # Adjust cache duration as desired.
    }
  }
}
```

## Reference on the headers

See the list of `x-authgear-` headers in the specs: [https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md](https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md)

