---
description: Integrate your website with Authgear JS SDK
---

# Web SDK

Authgear provides **token-based** or **cookie-based** authentication for web applications. You will need to decide which approach you are going to use before starting the setup.

### **Token-based authentication**

In token-based authentication, Authgear returns the `access token` and `refresh token` to the client app after authentication. 

The client SDK will automatically renew the `access token` with the `refresh token` for you, so you don't have to worry about it.

Your client app should call your backend with the access token on the Authorization header, and you can verify the access token by integrating Authgear with your backend.

This approach is suitable for **mobile apps** or **single-page web applications**.

Request example:

```bash
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> Authorization: Bearer <AUTHGEAR_ACCESS_TOKEN>
```

### **Cookie-based authentication**

In cookie-based authentication, Authgear returns `Set-Cookie` headers and sets cookies to the browser. The cookies are HTTP only and share under the same root domains. So you will need to setup the **custom domain** for Authgear, such as `identity.yourdomain.com`. 

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookies automatically. After that, you can verify the cookies by integrating Authgear with your backend. 

This approach is suitable for all types of websites, including server-side rendered applications.

Request example:

```javascript
> GET /api_path HTTP/1.1
> Host: yourdomain.com
> cookie: session=<AUTHGEAR_SESSION_ID>
```

## Setup Application in Authgear

Signup for an account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/) and create a project. Or you can use your self-deployed Authgear.

After that, we will need to create applications in Authgear.

{% tabs %}
{% tab title="Portal" %}
**Create OAuth Client**

1. Go to "Applications".
2. Click "Add Application" in the top right corner
3. Input name of your application, this is for reference only
4. Decide a path that users will be redirected to after they have authenticated with Authgear. Add the URI to "Redirect URIs". \(e.g. _https://yourdomain.com/auth-redirect_\).
5. If you want Authgear to issue JWT access token, select the "Issue JWT as access token". If you will use a reserve proxy to authenticate requests, keep this unchecked. Detail see [Integrate with your backend](backend-integration/).
6. \(Cookie-based authentication only\) If you are using cookies, you can decide the path that the user redirects to after logout. Add the URI to "Post Logout Redirect URIs".
7. Click "Save" and keep the client id. You can also obtain the client id from the list later.

**Add website to allowed origins**

1. Go to "Applications" &gt; "Allowed Origins".
2. Add your website origin to the allowed origins \(e.g. _yourdomain.com_, _localhost:4000_ for local development example\).
3. Click "Save".

**Setup custom domain for Authgear \(Required by cookie-based authentication only\)**

1. Go to "Custom Domains"
2. You will need to have a subdomain under your website so your users will log in via. \(e.g. _identity.yourdomain.com_\). In this case, a session cookie of domain "_yourdomain.com_" is written so that your websites can share the session cookies under "_yourdomain.com_" and all "_\*.yourdomain.com_" for better customer experiences.
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
    // make sure it is in the "Redirect URIs" list of the Application
    redirectURI: "https://yourdomain.com/auth-redirect",
    // prompt is optional
    // by default Authgear will not ask user to login again if user has logged in
    // if you want the user always reach the login page
    // and login again when you call startAuthorization
    // set promote to login like this
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

## Using the Access Token in HTTP Requests \(Token-based authentication only\)

If you are using cookies, you can skip this section.

To include the access token to the HTTP requests to your application server, there are two ways to achieve this.

### Option 1: Using fetch function provided by Authgear SDK

Authgear SDK provides the `fetch` function for you to call your application server. The `fetch` function will include the Authorization header in your application request, and handle refresh access token automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org/).

```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```

### Option 2: Add the access token to your HTTP

You can access the access token through `authgear.accessToken`. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of your application request.

```javascript
authgear
    .refreshAccessTokenIfNeeded()
    .then(() => {
        // access token is ready to use
        // accessToken can be string or undefined
        // it will be empty if user is not logged in or session is invalid
        const accessToken = authgear.accessToken;

        // include Authorization header in your application request
        const headers = {
            Authorization: `Bearer ${accessToken}`
        };
    });
```

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

## Next steps

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% page-ref page="backend-integration/" %}

