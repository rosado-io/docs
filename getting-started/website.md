---
description: Integrate your website with Authgear JS SDK
---

# Integrate with a Website

## Prerequisite

Signup for an account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/) and create a project. Or you can use your self-deployed Authgear.

After that, we will need to create applications in Authgear.

{% tabs %}
{% tab title="Portal" %}
**Create OAuth Client**

1. Go to "Applications".
2. Click "Add Application" in the top right corner
3. Input name of your application, this is for reference only
4. Decide a path that users will be redirected to after they have authenticated with Authgear. Add the URI to "Redirect URIs". \(e.g. https://yourdomain.com/auth-redirect\).
5. If you want Authgear to issue JWT access token, select the "Issue JWT as access token". If you will use a reserve proxy to authenticate requests, keep this unchecked. Detail see [Integrate with your backend](backend-integration/).
6. \(Optional\) If you are using cookies, you can decide the path that the user redirects to after logout. Add the URI to "Post Logout Redirect URIs".
7. Click "Save" and keep the client id. You can also obtain the client id from the list later.

**Add website to allowed origins**

1. Go to "Applications" &gt; "Allowed Origins".
2. Add your website origin to the allowed origins \(e.g. _yourdomain.com_, _localhost:4000_ for local development example\).
3. Click "Save".

**Setup custom domain for Authgear \(Required by using cookies only\)**

1. Go to "Custom Domains"
2. You will need to have a subdomain under your website so your users will log in via, e.g. "identity.yourdomain.com". In this case, a session cookie of domain "yourdomain.com" is written so that your websites can share the session cookies under "yourdomain.com" and all "\*.yourdomain.com" for better customer experiences.
3. Fill in your custom domain to the "Input New Domain" field and click "Add"; and follow the instruction to complete the custom domain setup.
4. Your custom domain status should be "Configured" after setup, click "Activate" to change it to "Active". The custom domain will be your new Authgear endpoint.
{% endtab %}

{% tab title="authgear.yaml \(self-deployed\)" %}
```text
http:
  allowed_origins:
    - "yourdomain.com"
    - "loclhost:4000" # example for local development
oauth:
  clients:
    - name: mywebsite
      client_id: a_random_generated_string
      redirect_uris:
        - "https://yourdomain.com/auth-redirect"
      grant_types:
        - authorization_code
        - refresh_token
      response_types:
        - code
        - none
```
{% endtab %}
{% endtabs %}

## Install the SDK

The Web JS SDK is available as [an npm package](https://www.npmjs.com/package/@authgear/web).

```bash
npm install --save --save-exact @authgear/web
```

## Configuration Authgear SDK

SDK must be properly configured before use.

```javascript
import authgear from "@authgear/web";

authgear
  .configure({
    // custom domain endpoint or default endpoint
    // default domain should be something like: https://<yourapp>.authgearapps.com
    endpoint: "<your_app_endpoint>",
    // client id of OAuth client
    clientID: "<your_api_key>",
    // sessionType support "refresh_token" or "cookie", default "refresh_token"
    // use "cookie" only if you plan to authenticate using cookie and
    // have custom domain setup
    sessionType: "refresh_token",
  })
  .then(
    () => {
      // configure successfully.
    },
    (err) => {
      // failed to configure.
    }
  );

```

## Trigger authenticate

When the user clicks login/signup on your website, you can use the following code to start authorization. 

```javascript
import authgear from "@authgear/web";

authgear
  .startAuthorization({
    // configure redirectURI which users will be redirected to
    // after they have authenticated with Authgear
    // you can use any path in your website
    // make sure it is in the "Redirect URIs" list of the OAuth client
    redirectURI: "https://yourdomain.com/auth-redirect",
    // prompt is optional
    // by default Authgear will not ask user to login again if user has logged in
    // Authgear and the session is still valid
    // if you want user always reach the login and login again when you call
    // startAuthorization, set promote to login like this
    prompt: "login",
  })
  .then(
    () => {
      // started authorization, user should be redirected to Authgear
    },
    (err) => {
      // failed to start authorization
    }
  );

```

## After user authenticate

After the user authenticates, the user will be redirected to the `redirectURI`. In your `redirectURI` , call the following code to complete authorization.

```javascript
import authgear from "@authgear/web";

authgear.finishAuthorization().then(
  (userInfo) => {
    // authorized successfully
  },
  (err) => {
    // failed to finish authorization
  },
);

```

Now, your user is now logged in! 

## Logout

If you are using `refresh_token` session type, `logout` API will revoke the user app session only. The promise will be resolved after logout and no redirection will occur. If you are using `cookie` , the user will be redirected to Authgear to log out Authgear session. You can provide `redirectURI` which user will be redirected to after logout.

```javascript
import authgear from "@authgear/web";

authgear
  .logout({
    // redirectURI is useful only when sessionType is cookie
    // users will be redirected to after logout
    // make sure it is in the "Post Logout Redirect URIs" list of the OAuth client
    redirectURI: "https://yourdomain.com",
  })
  .then(
    () => {
      // logged out successfully
    },
    (err) => {
      // failed to logout
    }
  );

```

## Secure your application server with Authgear

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear** and **using SDK to call your application server**. In the next section, we will go through the step one by one.

{% page-ref page="backend-integration/" %}





