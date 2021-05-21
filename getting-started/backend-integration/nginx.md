# Forward Authentication With Nginx

In this section, we will explain how to set up reverse proxy NGINX to protect your app server from unauthorized access with Authgear resolver.

## Overview of application request flow

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IENsaWVudEFwcCBhcyBVc2VyXG4gIHBhcnRpY2lwYW50IEFwcEJhY2tlbmQgYXMgWW91ciBBcHAgU2VydmVyIDxici8-IChXZWIgYmFja2VuZCwgQXBwIEFQSSBzZXJ2ZXIpXG4gIHBhcnRpY2lwYW50IEF1dGhnZWFyIGFzIEF1dGhnZXJcbiAgcmVjdCByZ2IoMjMwLCAyMzYsIDI0MSlcbiAgTm90ZSByaWdodCBvZiBDbGllbnRBcHA6IE1ha2luZyBhcHBsaWNhdGlvbiByZXF1ZXN0XG4gIENsaWVudEFwcC0-PkFwcEJhY2tlbmQ6IFNlbmRpbmcgYXBwbGljYXRpb24gcmVxdWVzdCB3aXRoIGFjY2VzcyB0b2tlbiAvIGNvb2tpZXNcbiAgQXBwQmFja2VuZC0-PkF1dGhnZWFyOiBTZXR1cCByZXZlcnNlIHByb3h5IHRvIGludGVncmF0ZSB3aXRoIEF1dGhnZWFyIHJlc29sdmVyIDxici8-IHRvIGF1dGhlbnRpY2F0ZSBIVFRQIHJlcXVlc3QgPGJyLz4gZS5nLiBuZ2lueCBuZ3hfaHR0cF9hdXRoX3JlcXVlc3RfbW9kdWxlXG4gIEF1dGhnZWFyLT4-QXBwQmFja2VuZDogQXV0aGdlYXIgcmVzb2x2ZXIgcmVzcG9uZHMgSFRUUCBoZWFkZXJzIDxici8-IGFuZCByZXZlcnNlIHByb3h5IHVwZGF0ZSB0aGUgcmVxdWVzdCA8YnIvPiBhcHAgc2VydmVyIHJlYWQgdGhlIGhlYWRlcnMgdG8gZGV0ZXJtaW5lIHVzZXIgbG9naW4gc3RhdGVcbiAgQXBwQmFja2VuZC0-PkNsaWVudEFwcDogUmVzcG9uc2UgdG8gdXNlclxuICBlbmQiLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCIsInNlcXVlbmNlIjp7InNob3dTZXF1ZW5jZU51bWJlcnMiOnRydWV9fSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)

1. After the user authenticated. Sending application request to your server with access token/cookies.
2. Set up a reserve proxy in your infrastructure to authenticate HTTP requests, it can be any application proxy that supports auth request. The proxy will request the Authgear resolver.
3. Authgear resolves parses the access token and returns HTTP headers with include user login state, the proxy will update the HTTP request with those headers. And you can read those headers in your application code to know who is the currently logged-in user.
4. Your app server returns the response to the client app. e.g. Returns user's content or HTTP 401 if the user is not logged in.

## How auth proxy works

For every request, your reverse proxy forwards the incoming HTTP request without the request body to the resolved endpoint of Authgear.

1. The HTTP response of the resolved endpoint will have headers starting with `x-authgear-`.
2. You have to instruct your reverse proxy to include those extra headers, before forwarding the request to your backend server.
3. Your backend server looks at the headers to determine whether the incoming HTTP request is authenticated or not.

There are so many reverse proxies available in the wild. So here we are going to illustrate the idea of using Nginx as the reverse proxy.

## Using Nginx as the reverse proxy

The trick here is to declare an internal `location` and use `auth_request` to initiate a subrequest to the resolved endpoint.

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

Authenticating every request could result in a huge downgrade in the performance, if the reverse proxy, Authgear, and your backend server are in different regions.

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

See [https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md](https://github.com/authgear/authgear-server/blob/master/docs/specs/api-resolver.md)

