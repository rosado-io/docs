# Integrate with your backend

## Overview

The integration is about letting your backend server know whether the incoming HTTP request is authenticated or not.
In order to do that, you have to setup a reverse proxy in front of your backend server.

1. For every request, your reverse proxy forwards the incoming HTTP request without the request body to the resolve endpoint of Authgear.
2. The HTTP response of the resolve endpoint will have headers starting with `x-authgear-`.
3. You have to instruct your reverse proxy to include those extra headers, before forwarding the request to your backend server.
4. Your backend server looks at the headers to determine whether the incoming HTTP request is authenticated or not.

There are so many reverse proxy available in the wild.
So here we are going to illustrate the idea using Nginx as the reverse proxy.

## Using Nginx as the reverse proxy

The trick here is to declare a internal `location` and use `auth_request` to initiate a subrequest to the resolve endpoint.

```text
server {
  # Use variable in proxy_pass with resolver to respect DNS TTL.
  # Note that /etc/hosts and /etc/resolv.conf are NOT consulted if resolver is used.
  # See https://www.nginx.com/blog/dns-service-discovery-nginx-plus/
  resolver 8.8.8.8;

  location / {
    set $backend http://www.mycompany.com;
    proxy_pass $backend;
    proxy_set_header Host $host;
    # Initiate a subrequest to the resolve endpoint.
    # This corresponds to the Step 1.
    # See http://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request
    auth_request /_auth;

    # Copy the `x-authgear-*` headers from the response of the subrequest to Nginx variables.
    # This corresponds to the Step 2.
    auth_request_set $x_authgear_session_valid $upstream_http_x_authgear_session_valid;
    auth_request_set $x_authgear_user_id $upstream_http_x_authgear_user_id;
    auth_request_set $x_authgear_user_anonymous $upstream_http_x_authgear_user_anonymous;
    auth_request_set $x_authgear_user_verified $upstream_http_x_authgear_user_verified;
    auth_request_set $x_authgear_session_acr $upstream_http_x_authgear_session_acr;
    auth_request_set $x_authgear_session_amr $upstream_http_x_authgear_session_amr;

    # Include the headers in the request that will be sent to your backend server.
    # This corresponds to the Step 3.
    proxy_set_header x-authgear-session-valid $x_authgear_session_valid;
    proxy_set_header x-authgear-user-id $x_authgear_user_id;
    proxy_set_header x-authgear-user-anonymous $x_authgear_user_anonymous;
    proxy_set_header x-authgear-user-verified $x_authgear_user_verified;
    proxy_set_header x-authgear-session-acr $x_authgear_session_acr;
    proxy_set_header x-authgear-session-amr $x_authgear_session_amr;

    # Your backend must inspect the request headers to determine whether the request is authenticated or not.
    # This corresponds to the Step 4.
  }

  location = /_auth {
    internal;
    set $backend https://mycompany.authgearapps.com/_resolver/resolve;
    proxy_pass $backend;
    # The body is supposed to be consumed by your backend server.
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
  }
}
```

## Optimizing the performance

Authenticating every request could result in hude downgrade in the performance if
the reverse proxy, Authgear and your backend server are in different regions.

You may consider enable caching.

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

See https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md
