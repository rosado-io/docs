---
description: How to integrate with a React Native app
---

# Integrate with a React Native app

This guide provides instructions to integrate authgear with a react native app. Supported platforms include:

* Android
* iOS

## Get the SDK

The React Native SDK is available as [a npm package](https://www.npmjs.com/package/@authgear/react-native).

## Prerequisite

You must follow [this](../deploy-on-your-cloud/local.md) to get Authgear running first!

## Add OAuth Client to Authgear

In [authgear.yaml](https://github.com/authgear/docs/tree/6a52b949b69188c9e946087c464c5b3b657b5bb4/configuration/authgear.yaml.md), declare an OAuth client for the app. This allows the app to authenticate via this client using the authgear sdk.

```yaml
oauth:
  clients:
    - client_id: a_random_generated_string
      redirect_uris:
        - "com.authgear.example://host/path"
      grant_types:
        - authorization_code
        - refresh_token
      response_types:
        - code
```

## Create a React Native app

Follow the documentation of React Native to see how you can create a new React Native app.

```bash
npx react-native init myapp
cd myapp
```

## Install the SDK

```bash
# The SDK depends on AsyncStorage.
yarn add --exact @react-native-community/async-storage
yarn add --exact @authgear/react-native
(cd ios && pod install)
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
        endpoint: "http://localhost:3000",
      })
      .then(() => {
        authgear
          .authorize({
            redirectURI: "com.authgear.example://host/path",
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

## Platform Integration

To finish the integration, setup the app to handle the redirectURI specified in previous section. This part requires platform specific integration.

### Android

Add the following activity entry to the `AndroidManifest.xml` of your app. The intent system would dispatch the redirect URI to `OAuthRedirectActivity` and the sdk would handle the rest.

```markup
<!-- Your application configuration. Omitted here for brevity -->
<application>
  <!-- Other activities or entries -->

  <!-- Add the following activity -->
  <activity android:name="com.oursky.authgear.reactnative.OAuthRedirectActivity"
            android:exported="false"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <!-- Configure data to be the exact redirect URI your app uses. -->
                <!-- Here, we are using com.authgear.example://host/path as configured in authgear.yaml. -->
                <!-- NOTE: The redirectURI supplied in AuthorizeOptions *has* to match as well -->
                <data android:scheme="com.authgear.example"
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
                              <string>com.authgear.example</string>
                      </array>
              </dict>
      </array>
</dict>
</plist>
```

#### Insert the SDK Integration Code

In `AppDelegate.m`, add the following code snippet.

```text
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
