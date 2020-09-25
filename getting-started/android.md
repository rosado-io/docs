---
description: How to use authgear android SDK
---

# Integrate with authgear android SDK

## Get the SDK

### Import the SDK module

1. Git clone `https://github.com/authgear/authgear-sdk-android.git`
2. In Android Studio, import the `sdk` module in the repository you just cloned via Files -> New -> Import module.

### Artifact

TODO: Support artifact

## Prerequisite

You must follow [this](local.md) to get Authgear running first!

## Add OAuth Client to Authgear

In [authgear.yaml](../configuration/authgear.yaml.md), declare an OAuth client for the app. This allows the app to authenticate via this client using the authgear sdk.

```yaml
oauth:
  clients:
  - client_id: a_random_generated_string
    redirect_uris:
    - "com.myapp://host/path"
    grant_types:
    - authorization_code
    - refresh_token
    response_types:
    - code
```

## Setup Redirect URI for Your Android App

Add the following activity entry to the `AndroidManifest.xml` of your app. The intent system would dispatch the redirect URI to `OauthRedirectActivity` and the sdk would handle the rest.

```xml
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
    private static final String AUTHGEAR_ENDPOINT = "http://localhost:3000/"
    private Authgear mAuthgear;
    public void onCreate() {
        super.onCreate();
        mAuthgear = new Authgear(this);
        mAuthgear.configure(new ConfigureOptions(CLIENT_ID, AUTHGEAR_ENDPOINT), new OnConfigureListener() {
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

Add the following code to your view model. Do *NOT* call these code in activity as this can lead to memory leak when your activity instance is destroyed. You can read more on view model in the official documentation [here](https://developer.android.com/topic/libraries/architecture/viewmodel).

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

The above call of `authorize` passes in the exact redirect uri as configured in `authgear.yaml` and manifest, the callback then indicates authorization success or failure. By default, the callback is called on the main thread.

## Kotlin coroutine support

The SDK comes with kotlin support out of the box. In fact, the SDK is written in kotlin!

If you are using kotlin, you can benefit from the suspend function APIs authgear provides. For example, the above authorize example can be written as follows:

```kotlin
import kotlinx.coroutines.*

class MyAwesomeViewModel(application: MyAwesomeApplication) : AndroidViewModel(application) {
    // Other methods

    // This is called when login button is clicked.
    fun login() {
        viewModelScope {
            withContext(Dispatchers.IO) {
                try {
                    val app = getApplication<MyAwesomeApplication>()
                    val state = app.authgear.authorize(AuthorizeOptions(redirectUri = "com.myapp://host/path"))
                    // User is logged in!
                } catch (e: Throwable) {
                    // Something went wrong.
                }
            }
        }
    }
}
```

## Using the Access Token in HTTP Requests

Since there are many libraries and patterns to perform HTTP requests in android, authgear does *not* provide a centralized request client. Users are encouraged to integrate authgear with their library of choice using the functionalities provided by authgear. Theese functionaltities involve two main parts: getting the access token for use in bearer token, and refreshing the access token when necessary.

The following code demonstrate how this is achieved:

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
    // It is up to the caller to decide how to handle this situation. Typically, the request could be aborted
    // immediately as the response would be 401 anyways.
    return;
}

HashMap<String, String> headers = new HashMap<>();
headers.put("authorization", "bearer " + accessToken);

// Submit the request with the headers...
```

The first line of the code handles all logic to check access token validity and refresh the access token when necessary. The method suffix `Sync` indicates this is a blocking call and thus the access token obtained in the second line is guaranteed to be valid as long as it is not null. The access token can then be used in the `authorization` header.

It is up to the user to decide how to handle the scenario in which the user is not logged in or the refresh token is expired.

## OKHttp Interceptor Extension (Optional)

The SDK provides an optional `Okhttp` interceptor which handles everything from refreshing the access token to putting the access token in the header.

### Get the Extension

1. Git clone the SDK repository. If you have not already, refer to the instructions above.
2. In Android Studio, import the `sdk-okhttp` module in the repository you just cloned via Files -> New -> Import module.

### Usage

Configure `OkHttpClient` to use `AuthgearInterceptor` as follows:

```java
Authgear authgear = // Obtain the authgear instance.
OKHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(AuthgearInterceptor(authgear))
            .build()
```

The client would then include the access token in every request, and refresh the access token when necessary before the requests.
