---
description: How to integrate with a React Native app
---

# React Native SDK

This guide provides instructions to integrate authgear with a react native app. Supported platforms include:

* Android
* iOS

## Setup Application in Authgear

Signup for an Authgear Portal account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/). Or you can use your self-deployed Authgear.

From the Project listing, create a new Project or select an existing Project. After that, we will need to create an application in the project.

{% tabs %}
{% tab title="Portal" %}
**Step 1: Create an application in the Portal**

1. Go to **Applications** on the left menu bar.
1. Click **⊕Add Application** in the top tool bar.
1. Input the name of your application and select **Native App** as the application type. Click "Save".
1. You will see a list of guides that can help you for setting up, then click "Next".

![Create an application](../.gitbook/assets/create-application-app-1.png)

**Step 2: Configure the application**

1. In your IDE, define a custom URI scheme that the users will be redirected back to your app after they have authenticated with Authgear, e.g. `com.myapp.example://host/path`.\[^1]
1. Head back to Authgear Portal, fill in the URI that you have defined in the previous steps.
1. Click "Save" in the top tool bar and keep the **Client ID**. You can also obtain it again from the Applications list later.

![Edit an application](../.gitbook/assets/edit-application-app.png)

{% hint style="info" %}
If you wish to [validate JSON Web Token (JWT) in your own application server](../backend-integration/jwt/), turn on "Issue JWT as access token".\[^2] If you wish to [forward authentication requests to Authgear Resolver Endpoint](../backend-integration/nginx/), leave this unchecked. See comparisons in [Backend Integration](../backend-integration/).
{% endhint %}

![](../.gitbook/assets/application-jwt.png)

{% endtab %}
{% tab title="authgear.yaml (self-deployed)" %}
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

## Create a React Native app

Follow the documentation of React Native to see how you can create a new React Native app.

```bash
npx react-native init myapp
cd myapp
```

## Install the SDK

```bash
yarn add --exact @authgear/react-native
(cd ios && pod install)
```

## Platform Integration

To finish the integration, setup the app to handle the redirectURI specified in the application. This part requires platform specific integration.

### Android

Add the following activity entry to the `AndroidManifest.xml` of your app. The intent system would dispatch the redirect URI to `OAuthRedirectActivity` and the sdk would handle the rest.

```markup
<!-- Your application configuration. Omitted here for brevity -->
<application>
  <!-- Other activities or entries -->

  <!-- Add the following activity -->
  <activity android:name="com.authgear.reactnative.OAuthRedirectActivity"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <!-- Configure data to be the exact redirect URI your app uses. -->
                <!-- Here, we are using com.authgear.example://host/path as configured in authgear.yaml. -->
                <!-- NOTE: The redirectURI supplied in AuthorizeOptions *has* to match as well -->
                <data android:scheme="com.myapp.example"
                    android:host="host"
                    android:pathPrefix="/path"/>
            </intent-filter>
  </activity>
</application>
```

### iOS

#### Declare URL Handling in Info.plist

In `Info.plist`, add the matching redirect URI.

```markup
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
      <!-- Other entries -->
      <key>CFBundleURLTypes</key>
      <array>
              <dict>
                      <key>CFBundleTypeRole</key>
                      <string>Editor</string>
                      <key>CFBundleURLSchemes</key>
                      <array>
                              <string>com.myapp.example</string>
                      </array>
              </dict>
      </array>
</dict>
</plist>
```

#### Insert the SDK Integration Code

In `AppDelegate.m`, add the following code snippet.

```objectivec
// Other imports...
#import <authgear-react-native/AGAuthgearReactNative.h>

// Other methods...

// For handling deeplink
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:
                (NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
    return [AGAuthgearReactNative application:app openURL:url options:options];
}

// For handling deeplink
// deprecated, for supporting older devices (iOS < 9.0)
- (BOOL)application:(UIApplication *)application
              openURL:(NSURL *)url
    sourceApplication:(NSString *)sourceApplication
          annotation:(id)annotation {
    return [AGAuthgearReactNative application:application
                                      openURL:url
                            sourceApplication:sourceApplication
                                  annotation:annotation];
}

// for handling universal link
- (BOOL)application:(UIApplication *)application
    continueUserActivity:(NSUserActivity *)userActivity
      restorationHandler:
          (void (^)(NSArray<id<UIUserActivityRestoring>> *_Nullable))
              restorationHandler {
    return [AGAuthgearReactNative application:application
                        continueUserActivity:userActivity
                          restorationHandler:restorationHandler];
}
```

## Try authenticate

Add this code to your react native app. This snippet configures authgear to connect to an authgear server deployed at `endpoint` with the client you have just setup via `clientID`, opens a browser for authorization, and then upon success redirects to the app via the `redirectURI` specified.

```javascript
import React, { useCallback } from "react";
import { View, Button } from "react-native";
import authgear from "@authgear/react-native";

function LoginScreen() {
  const onPress = useCallback(() => {
    // Normally you should only configure once when the app launches.
    authgear
      .configure({
        clientID: "a_random_generated_string",
        endpoint: "http://<myapp>.authgearapps.com",
      })
      .then(() => {
        authgear
          .authenticate({
            redirectURI: "com.myapp.example://host/path",
          })
          .then(({ userInfo }) => {
            console.log(userInfo);
          });
      });
  }, []);

  return (
    <View>
      <Button onPress={onPress} title="Authenticate" />
    </View>
  );
}
```

## Using the Access Token in HTTP Requests

To include the access token to the HTTP requests to your application server, there are two ways to achieve this.

### Option 1: Using fetch function provided by Authgear SDK

Authgear SDK provides the `fetch` function for you to call your application server. The `fetch` function will include the Authorization header in your application request, and handle refresh access token automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org).

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

To log out the user from the current app session, you need to invoke the`logout`function.

```javascript
authgear
  .logout()
  .then(() => {
    // logout successfully
  });
```

## Next steps

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% content-ref url="backend-integration/" %}
[backend-integration](backend-integration/)
{% endcontent-ref %}

## JavaScript SDK Reference

For detailed documentation on the JavaScript React Native SDK, visit [@authgear/react-native Reference](https://authgear.github.io/authgear-sdk-js/docs/react-native/)

### Footnote

\[^1]: For futher instruction on setting up custom URI scheme in React Native, see [https://reactnative.dev/docs/linking](https://reactnative.dev/docs/linking) \[^2]: For more explaination on JWT, see [https://en.wikipedia.org/wiki/JSON\_Web\_Token](https://en.wikipedia.org/wiki/JSON\_Web\_Token)
