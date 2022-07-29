---
description: Integrate Authgear to your website with the Web SDK
---

# Web SDK

## Setup Application in Authgear

Signup for an Authgear Portal account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/). Or you can use your self-deployed Authgear.

From the Project listing, create a new Project or select an existing Project. After that, we will need to create an application in the project.

{% tabs %}
{% tab title="Portal" %}
**Step 1: Create an application in the Portal**

1. Go to **Applications** on the left menu bar.
2. Click **⊕Add Application** in the top tool bar.
3. Input the name of your application, e.g. "MyAwesomeApp".
4. Under **Authorized Redirect URIs**, Click "⊕Add URI".
5. Decide a path in your website that users will be redirected to after they have authenticated with Authgear. e.g.`https://yourdomain.com/auth-redirect` , or `http://localhost:4000/auth-redirect` for local development.
6. Fill in the Redirect URI that you have defined in the previous steps.
7. Click "Save" in the top tool bar and keep the **Client ID**. You can also obtain it again from the Applications list later.
8. (Optional) Click "Edit" if you wish to configure more authentication settings.

![](<../.gitbook/assets/create-application (1).png>)

**Step 2: Configure the application**

1. In **Edit Application**, if you are using [cookie-based authentication](authentication-approach/cookie-based.md), you can decide the path that the user redirects to after logout. Add the URI to **Post Logout Redirect URIs.**
2. If you want to validate JWT access token in your server, under **Token Settings**, select **Issue JWT as access token**. If you will forward incoming requests to Authgear Resolver Endpoint for authentication, leave this unchecked. See comparisons in [Backend Integration](backend-integration/).

![](<../.gitbook/assets/edit-application (1).png>)

**Step 3: Add your website to allowed origins**

1. Go to **Applications** > **Allowed Origins**.
2. Add your website origin to the allowed origins. e.g. `yourdomain.com` or `localhost:4000` for local development.
3. Click "Save".

**Step 4: Setup a custom domain (Required for** [**Cookie-based authentication**](authentication-approach/cookie-based.md)**)**

1. Go to **Custom Domains**
2. Add your custom domain in **Input New Domain**
3. Follow the instructions to verify and configure the custom domain.
4. Click "Activate" to change the domain status to "Active". The custom domain will be your new Authgear endpoint.

{% hint style="info" %}
For cookie-based authentication, users will log in via the custom domain e.g. _"auth.yourdomain.com"_ . So that a session cookie of your domain is set for the client and all your websites under _"\*.yourdomain.com"_ would share the same session cookies automatically.
{% endhint %}
{% endtab %}

{% tab title="authgear.yaml (self-deployed)" %}
```
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

## Install the Authgear Web SDK

The Web JS SDK is available as [an npm package](https://www.npmjs.com/package/@authgear/web). Install the package in your project directory

#### NPM

```bash
npm install --save --save-exact @authgear/web
```

#### Yarn

```bash
yarn add @authgear/web --exact
```

## Initialize the Authgear SDK

The SDK must be properly configured before use. Call the `configure` method every time your page loads up.

```javascript
import authgear from "@authgear/web";

authgear
  .configure({
    // custom domain endpoint or default endpoint
    // default domain should be something like: https://<yourapp>.authgearapps.com
    endpoint: "<your_app_endpoint>",
    // Client ID of your application
    clientID: "<your_api_key>",
    // sessionType can be "refresh_token" or "cookie", default "refresh_token"
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

The SDK should be [configured](https://authgear.github.io/authgear-sdk-js/docs/web/interfaces/configureoptions) according to the authentication approach of your choice

* **Token-based authentication**
  * `sessionType` should be set to `refresh_token`
* **Cookie-based authentication**
  * `endpoint` must be a custom domain endpoint
  * `sessionType` should be set to `cookie`

## Login to the application

### Step 1: Start the auth flow

When the user clicks login/signup on your website, make a **start authorization** call to redirect them to the login/signup page.

```javascript
import authgear from "@authgear/web";

authgear
  .startAuthentication({
    // configure redirectURI which users will be redirected to
    // after they have authenticated with Authgear
    // you can use any path in your website
    // make sure it is in the "Redirect URIs" list of the Application
    redirectURI: "https://yourdomain.com/auth-redirect",
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

By default, Authgear will not ask user to login again if user has already logged in. You can optionally set `prompt` to `login` if you want the user always reach the login page and login again.

### Step 2: Handling auth result in the redirectURI

After the user authenticates on the login page, the user will be redirected to the `redirectURI` with a `code` parameter in the URL query. In the `redirectURI` of your application, make a **finish authorization** call to handle the authentication result. This will attempt to exchange the `code` for the access token and user info.

Once authorization succeed, the application should navigate the user to other URL such as the user's home page and remove the `code` query parameters.

```javascript
import authgear from "@authgear/web";

authgear.finishAuthentication().then(
  (userInfo) => {
    // authorized successfully
    // you should navigate the user to another path
  },
  (err) => {
    // failed to finish authorization
  },
);
```

Now, your user is logged in!

## Get the Logged In State

The `sessionState` reflects the user logged in state in the SDK local state. That means even the`sessionState` is `AUTHENTICATED`, the session may be invalid if it is revoked remotely. After initializing the Authgear SDK, call `fetchUserInfo` to update the `sessionState` as soon as it is proper to do so.

```javascript
// value can be NO_SESSION or AUTHENTICATED
// After authgear.configure, it only reflect SDK local state.
let sessionState = authgear.sessionState;

if (sessionState === "AUTHENTICATED") {
    authgear
        .fetchUserInfo()
        .then((userInfo) => {
            // sessionState is now up to date
        })
        .catch((e) => {
            // sessionState is now up to date
            // it will change to NO_SESSION if the session is invalid
        });
}
```

The [`UserInfo`](../integrate/user-profile.md) object provides the unique identifier of the user from your Authgear project. See more details [here](../integrate/user-profile.md).

## Log the user out

Use the `logout` function to log out the user. The user will be redirected to your Authgear endpoint to log out their session. You should provide the `redirectURI` for the user to return to your app.

```javascript
import authgear from "@authgear/web";

authgear
  .logout({
    // user will navigate to the redirectURI after logged out
    // make sure it is in the "Post Logout Redirect URIs" in the application portal
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

## Calling an API

### Token-based authentication

To include the access token to the HTTP requests to your application server, there are two ways to achieve this.

#### Option 1: Using fetch function provided by Authgear SDK

Authgear SDK provides the `fetch` function for you to call your application server. This `fetch` function will include the Authorization header in your application request, and handle refresh access token automatically. The `authgear.fetch` implements [fetch](https://fetch.spec.whatwg.org/).

```javascript
authgear
    .fetch("YOUR_SERVER_URL")
    .then(response => response.json())
    .then(data => console.log(data));
```

#### Option 2: Add the access token to the HTTP request header

You can get the access token through `authgear.accessToken`. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of the application request.

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

### Cookie-based authentication

If you are using cookies, all requests from your applications under `*.yourdomain.com` to your application server will include the session cookie automatically. You can skip this section and see the next step: [Backend Integration](backend-integration/)

## Next steps

To protect your application server from unauthorized access. You will need to **integrate Authgear to your backend**.

{% content-ref url="backend-integration/" %}
[backend-integration](backend-integration/)
{% endcontent-ref %}

## JavaScript SDK Reference

For detailed documentation on the JavaScript Web SDK, visit [@authgear/web Reference](https://authgear.github.io/authgear-sdk-js/docs/web/)
