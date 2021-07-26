# Local Development Setup for cookie-based Authorization

[Running a local Authgear server](../../deploy-on-your-cloud/local.md) is not trivial. This guide provides a simple way to bootstrap your local application that communicates with the production Authgear server.

{% hint style="warning" %}
This page is only for **cookie-based** authorization in **local development** setup. **DO NOT** follow this guide if you are using token-based authorization or setting up your production environment.
{% endhint %}

## Problems using localhost as local website domain

Authgear sets cookie in the browser inside the project domain. When running your application on `localhost`, the browser will not see the cookies because the website is not in the subdomain of the domain in which the cookies are set. Therefore, the browser will not be able to authenticate itself.

You can learn more [here](../../get-started/authentication-approach/cookie-based.md#how-it-works).

## Setup a new Authgear project

> For local development, it is highly recommended to create a new application on Authgear before continuing to the rest of the guide.

1. Log in and create a new project on [https://portal.authgearapps.com/projects](https://portal.authgearapps.com)
2. Go to the **Application** tab in your dashboard
3. Add your local application domain `{SUBDOMAIN}.{PROJECT_NAME}.authgearapps.com` under the **Allowed Origins** list 
4. Add an application, name it whatever you want. **DO NOT check the Issue JWT as access token** box because we are using cookie-based authorization. 
5. Put your redirect URI for login and logout under the **Redirect URIs** list and **Post Logout Redirect URIs** list respectively.

## Map domain in `hosts`

To make the cookies visible to the browser, the local website domain has to be inside the domain where the cookies are set.

You can fake the browser that you are using a subdomain of the project domain by manually mapping our local application domain to your loopback address in the `hosts` file, adding the following line:
```
127.0.0.1 {SUBDOMAIN}.{PROJECT_NAME}.authgearapps.com
```

The browser will be able to see the auth cookies if visiting the website via this domain.

## Use HTTPS
Although you can see the cookies now, the cookies have the **Secure** attribute set. To include them in an HTTP request, the request has to be transmitted over a secure channel (**HTTPS** in most browsers). Therefore, we also need to establish **HTTPS** connections for our browser with the server.

### Generate certificates
One quick simple way to do this is to use [mkcert](https://github.com/FiloSottile/mkcert), you may follow the installation steps [here](https://github.com/FiloSottile/mkcert#installation). After installing mkcert, generate a certificate with the following command:
```
mkcert *.{PROJECT_NAME}.authgearapps.com
```

A key file and a cert file will be generated. They will be used in the next part of the guide.

### Using nginx

We will need an **nginx** server to serve the certificate and enable SSL.

Add the following config file to your `nginx/conf.d` directory, or mount it to a volume together with the cert and key if you are using nginx in docker. 

Below examples show the nginx config files for nginx in host and nginx in docker.

{% tabs %}
{% tab title="nginx in host" %}
```
server {
  listen       443 ssl;
  server_name  {SUBDOMAIN}.{PROJECT_NAME}.authgearapps.com;
  
  ssl_certificate      /path/to/your/cert;
  ssl_certificate_key  /path/to/your/key;

  location / {
    # Change it to your service endpoint
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;

    auth_request /_auth;
    auth_request_set $x_authgear_session_valid $upstream_http_x_authgear_session_valid;
    auth_request_set $x_authgear_user_id $upstream_http_x_authgear_user_id;
    auth_request_set $x_authgear_user_anonymous $upstream_http_x_authgear_user_anonymous;
    auth_request_set $x_authgear_user_verified $upstream_http_x_authgear_user_verified;
    auth_request_set $x_authgear_session_acr $upstream_http_x_authgear_session_acr;
    auth_request_set $x_authgear_session_amr $upstream_http_x_authgear_session_amr;

    proxy_set_header x-authgear-session-valid $x_authgear_session_valid;
    proxy_set_header x-authgear-user-id $x_authgear_user_id;
    proxy_set_header x-authgear-user-anonymous $x_authgear_user_anonymous;
    proxy_set_header x-authgear-user-verified $x_authgear_user_verified;
    proxy_set_header x-authgear-session-acr $x_authgear_session_acr;
    proxy_set_header x-authgear-session-amr $x_authgear_session_amr;
  }

  location /_auth {
    internal;
    resolver 8.8.8.8;
    set $resolver https://getchatbutton.authgearapps.com/_resolver/resolve;
    proxy_pass $resolver;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
  }
}
```
{% endtab %}
{% tab title="nginx in docker" %}
{% hint style="info" %}
Docker `host` network driver is not supported in Docker Desktop, it has to be in a `bridge` network. If your nginx in docker needs to proxy requests to services in your host network, it needs to resolve `host.docker.intenrnal` through `127.0.0.11`. If your services are also in the same docker bridge network (i.e. same docker-compose without specifying multiple networks), the destination domain will be the container name.
{% endhint %}

```
server {
  listen       443 ssl;
  server_name  {SUBDOMAIN}.{PROJECT_NAME}.authgearapps.com;
  
  ssl_certificate      /path/to/your/cert;
  ssl_certificate_key  /path/to/your/key;

  location / {
    resolver 127.0.0.11;
    # change {CONTAINER_NAME} to host.docker.internal if accessing host
    proxy_pass http://{CONTAINER_NAME}:{PORT};
    proxy_set_header Host $host;

    auth_request /_auth;
    auth_request_set $x_authgear_session_valid $upstream_http_x_authgear_session_valid;
    auth_request_set $x_authgear_user_id $upstream_http_x_authgear_user_id;
    auth_request_set $x_authgear_user_anonymous $upstream_http_x_authgear_user_anonymous;
    auth_request_set $x_authgear_user_verified $upstream_http_x_authgear_user_verified;
    auth_request_set $x_authgear_session_acr $upstream_http_x_authgear_session_acr;
    auth_request_set $x_authgear_session_amr $upstream_http_x_authgear_session_amr;

    proxy_set_header x-authgear-session-valid $x_authgear_session_valid;
    proxy_set_header x-authgear-user-id $x_authgear_user_id;
    proxy_set_header x-authgear-user-anonymous $x_authgear_user_anonymous;
    proxy_set_header x-authgear-user-verified $x_authgear_user_verified;
    proxy_set_header x-authgear-session-acr $x_authgear_session_acr;
    proxy_set_header x-authgear-session-amr $x_authgear_session_amr;
  }

  location /_auth {
    internal;
    resolver 8.8.8.8;
    set $resolver https://getchatbutton.authgearapps.com/_resolver/resolve;
    proxy_pass $resolver;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
  }
}
```
{% endtab %}
{% endtabs %}

In the above examples, nginx will also authenticate requests by creating sub-requests to the Authgear internal endpoint. You can learn more [here](https://docs.authgear.com/deploy-on-your-cloud/auth-nginx#add-nginx).

## Finish

Now visit the website through `https://{SUBDOMAIN}.{PROJECT_NAME}.authgearapps.com`, the browser will be able to send requests with the authorization cookies.

For implementing login and logout logic in your website, please refer to [Web SDK](../../get-started/website.md).