---
description: How to use authgear android SDK
---

# Android SDK

## Setup Application in Authgear

Signup for an account in [https://portal.authgearapps.com/](https://portal.authgearapps.com) and create a project. Or you can use your self-deployed Authgear.

After that, we will need to create an application in Authgear.

{% tabs %}
{% tab title="Portal" %}
**Create an application**

1. Go to "Applications".
2. Click "Add Application" in the top right corner
3. Input name of your application, this is for reference only
4. Defining a custom scheme that the users will be redirected back to your app after they have authenticated with Authgear. Add the URI to "Redirect URIs". (e.g. _com.myapp://host/path_).
5. Click "Save" and keep the client id. You can also obtain the client id from the list later.

{% hint style="info" %}
If you want to validate JWT access token in your server, select **Issue JWT as access token**. If you will forward incoming requests to Authgear Resolver Endpoint for authentication, leave this unchecked. See comparisons in [Backend Integration](../backend-integration/).
{% endhint %}
{% endtab %}

{% tab title=" authgear.yaml (self-deployed)" %}
```yaml
oauth:
  clients:
    - name: your_app_name
      client_id: a_random_generated_string
      redirect_uris:
        - "com.myapp://host/path"
      grant_types:
        - authorization_code
        - refresh_token
      response_types:
        - code
        - none
```
{% endtab %}
{% endtabs %}

## Get the SDK

1.  Add `jitpack` repository to `gradle`

    ```
    allprojects {
        repositories {
            // Other repository
            maven { url 'https://jitpack.io' }
        }
    }
    ```
2.  Add authgear in dependencies. Use `$branch-SNAPSHOT` (e.g. `main-SNAPSHOT`) for the latest version in a branch or a release tag/git commit hash of the desired version.

    ```
    dependencies {
        // Other implementations
        implementation 'com.github.authgear:authgear-sdk-android:SNAPSHOT'
    }
    ```

## Setup Redirect URI for Your Android App

Add the following activity entry to the `AndroidManifest.xml` of your app. The intent system would dispatch the redirect URI to `OauthRedirectActivity` and the SDK would handle the rest.

```markup
<!-- Your application configuration. Omitted here for brevity -->
<application>
  <!-- Other activities or entries -->

  <!-- Add the following activity -->
  <activity android:name="com.oursky.authgear.OauthRedirectActivity"
            android:exported="false"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <!-- Configure data to be the exact redirect URI your app uses. -->
                <!-- Here, we are using com.myapp://host/path as configured in authgear.yaml. -->
                <!-- NOTE: The redirectURI supplied in AuthorizeOptions *has* to match as well -->
                <data android:scheme="com.myapp"
                    android:host="host"
                    android:pathPrefix="/path"/>
            </intent-filter>
  </activity>
</application>
```

## Initialize Authgear

Add the following code to your app's `Application` class. If there is none, add a class that extends `Application`. Make sure it is declared in `AndroidManifest.xml`'s `application` tag with the `name` attribute as described [here](https://developer.android.com/guide/topics/manifest/application-element#nm).

```java
public class MyAwesomeApplication extends Application {
    // The client ID of the oauth client.
    private static final String CLIENT_ID = "a_random_generated_string"
    // Deployed authgear's endpoint
    private static final String AUTHGEAR_ENDPOINT = "http://<myapp>.authgearapps.com/"
    private Authgear mAuthgear;
    public void onCreate() {
        super.onCreate();
        mAuthgear = new Authgear(this, CLIENT_ID, AUTHGEAR_ENDPOINT);
        mAuthgear.configure(new OnConfigureListener() {
            @Override
            public void onConfigured() {
                // Authgear can be used.
            }

            @Override
            public void onConfigurationFailed(@NonNull Throwable throwable) {
                Log.d(TAG, throwable.toString());
                // Something went wrong, check the client ID or endpoint.
            }
        });
    }

    public Authgear getAuthgear() {
        return mAuthgear;
    }
}
```

## Authorize a user

Add the following code to your view model. Do _NOT_ call these codes in activity as this can lead to memory leak when your activity instance is destroyed. You can read more on the view model in the official documentation [here](https://developer.android.com/topic/libraries/architecture/viewmodel).

```java
class MyAwesomeViewModel extends AndroidViewModel {
    // Other methods

    // This is called when login button is clicked.
    public void login() {
        MyAwesomeApplication app = getApplication();
        app.getAuthgear().authorize(new AuthorizeOptions(
                "com.myapp://host/path",
                null,
                null,
                null,
                null
        ), new OnAuthorizeListener() {
            @Override
            public void onAuthorized(@Nullable String state) {
                // The user is logged in!
            }

            @Override
            public void onAuthorizationFailed(@NonNull Throwable throwable) {
                // Something went wrong.
            }
        });
    }
}
```

The above call of `authorize` passes in the exact redirect URI as configured in the applications and manifest, the callback then indicates authorization success or failure. By default, the callback is called on the main thread.

## Using the Access Token in HTTP Requests

Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make the network call only if the access token has expired. Include the access token into the Authorization header of your application request. If you are using OKHttp in your project, you can also use the interceptor extension provided by the SDK, see [detail](okhttp-interceptor-extension.md).

```java
// Suppose we are preparing an http request in a background thread.

// Setting up the request, e.g. preparing a URLConnection

try {
    authgear.refreshAccessTokenIfNeededSync();
} catch (OauthException e) {
    // The token is expired.
}
String accessToken = authgear.getAccessToken();
if (accessToken == null) {
    // The user is not logged in, or the token is expired.
    // It is up to the caller to decide how to handle this situation.
    // Typically, the request could be aborted
    // immediately as the response would be 401 anyways.
    return;
}

HashMap<String, String> headers = new HashMap<>();
headers.put("authorization", "Bearer " + accessToken);

// Submit the request with the headers...
```

## Logout

To log out the user from the current app session, you need to invoke the`logout`function.

```java
class MyAwesomeViewModel extends AndroidViewModel {
    // Other methods

    // This is called when logout button is clicked.
    public void logout() {
        MyAwesomeApplication app = getApplication();
        app.getAuthgear().logout(.logout(new OnLogoutListener() {
            @Override
            public void onLogout() {
                // Logout successfully
            }

            @Override
            public void onLogoutFailed(@NonNull Throwable throwable) {
                // Failed to logout
            }
        });
    }
}
```

## **Next steps** <a href="secure-your-application-server-with-authgear" id="secure-your-application-server-with-authgear"></a>

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% content-ref url="../backend-integration/" %}
[backend-integration](../backend-integration/)
{% endcontent-ref %}
