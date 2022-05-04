---
description: How to integrate with a Flutter app
---

# Flutter SDK

This guide provides instructions to integrate Authgear with a Flutter app. Supported platforms include:

* Android
* iOS

## Setup Application in Authgear

Signup for an account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/) and create a project. Or you can use your self-deployed Authgear.

After that, we will need to create an application in Authgear.

{% tabs %}
{% tab title="Portal" %}
**Create an application**

1. Go to "Applications".
2. Click "Add Application" in the top right corner
3. Input the name of your application, this is for reference only
4. Defining a custom scheme that the users will be redirected back to your app after they have authenticated with Authgear. Add the URI to "Redirect URIs". \(e.g. _com.myapp://host/path_\).
5. Click "Save" and keep the client id. You can also obtain the client id from the list later.

{% hint style="info" %}
If you want to validate JWT access token in your server, select **Issue JWT as access token**. If you will forward incoming requests to Authgear Resolver Endpoint for authentication, leave this unchecked. See comparisons in [Backend Integration](backend-integration/).
{% endhint %}
{% endtab %}

{% tab title="authgear.yaml \(self-deployed\)" %}
```javascript
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

## Create a Flutter app

Follow the documentation of Flutter to see how you can create a new Flutter app.

```bash
flutter create myapp
cd myapp
```

## Install the SDK

```bash
flutter pub add flutter_authgear
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
  <activity android:name="com.authgear.flutter.OAuthRedirectActivity"
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

## Try authenticate

Add this code to your app. This snippet configures authgear to connect to an authgear server deployed at `endpoint` with the client you have just setup via `clientID`, opens a browser for authorization, and then upon success redirects to the app via the `redirectURI` specified.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_authgear/authgear.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}


class _MyAppState extends State<MyApp> {
  late Authgear _authgear;

  @override
  void initState() {
    super.initState();
    _init();
  }

  Future<void> _init() {
    _authgear = Authgear(endpoint: "ENDPOINT", clientID: "CLIENT_ID");
    await _authgear.configure();
  }

  @override
  Widget build(BuildContext context) {
    return TextButton(
      child: Text("Authenticate"),
      onPressed: _onPressedAuthenticate,
    );
  }

  _Future<void> _onPressedAuthenticate() {
    try {
      await _authgear.authenticate(redirectURI: "REDIRECT_URI");
    } on CancelException {
    }
  }
}
```

## Using the Access Token in HTTP Requests

To include the access token to the HTTP requests to your application server, use `wrapHttpClient`.

The wrapped client will include the Authorization header in every HTTP request, and refresh access token automatically.

```dart
final originalClient = ...
final client = authgear.wrapHttpClient(originalClient);
```

## Logout

To log out the user from the current app session, you need to invoke the`logout`function.

```dart
await authgear.logout();
```

## Next steps

To protect your application server from unauthorized access. You will need to **integrate your backend with Authgear**.

{% page-ref page="backend-integration/" %}

