---
description: How to integrate with a React Native app
---

# Integrate with a React Native app

> TODO: Add instruction for Android

## Integrate with a React Native app

The React Native SDK is available as [a npm package](https://www.npmjs.com/package/@authgear/react-native).

### Prerequisite

You must follow [this](local.md) to get Authgear running first!

### Create a React Native app

Follow the documentation of React Native to see how you can create a new React Native app.

```bash
npx react-native init myapp
cd myapp
```

### Install the SDK

```bash
# The SDK depends on AsyncStorage.
yarn add --exact @react-native-community/async-storage
yarn add --exact @authgear/react-native
(cd ios && pod install)
```

### Add OAuth client

In [authgear.yaml](../configuration/authgear.yaml.md), declare an OAuth client for the app.

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
                              <string>com.myapp</string>
                      </array>
              </dict>
      </array>
</dict>
</plist>
```

### Insert the SDK integration code

In `AppDelegate.m`, add the following code snippet.

```text
// Other imports...
#import <authgear-react-native/AGAuthgearReactNative.h>

// Other methods...

- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:
                (NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
    return [AGAuthgearReactNative application:app openURL:url options:options];
}

- (BOOL)application:(UIApplication *)application
              openURL:(NSURL *)url
    sourceApplication:(NSString *)sourceApplication
          annotation:(id)annotation {
    return [AGAuthgearReactNative application:application
                                      openURL:url
                            sourceApplication:sourceApplication
                                  annotation:annotation];
}

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

### Try authenticate

Sample code.

```javascript
import React, { useCallback } from "react";
import { View, Button } from "react-native";
import authgear from "@authgear/react-native";

function LoginScreen() {
  const onPress = useCallback(() => {
    // Normally you should only configure once when the app launches.
    authgear.configure({
      clientID: "a_random_generated_string",
      endpoint: "http://localhost:3000",
    }).then(() => {
      authgear.authorize({
        redirectURI: "com.myapp://host/path",
      }).then(({ userInfo }) => {
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

