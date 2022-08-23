---
description: How to integrate with a Flutter app
---

# Flutter SDK

This guide provides instructions to integrate Authgear with a Flutter app. Supported platforms include:

* Android
* iOS

## Setup Application in Authgear

Signup for an Authgear Portal account in [https://portal.authgearapps.com/](https://portal.authgearapps.com/). Or you can use your self-deployed Authgear.

From the Project listing, create a new Project or select an existing Project. After that, we will need to create an application in the project.

{% tabs %}
{% tab title="Portal" %}
**Create an application**

1. Go to **Applications** on the left menu bar.
2. Click **⊕Add Application** in the top tool bar.
3. Input the name of your application and select **Native App** as the application type. Click "Save".

![](<../.gitbook/assets/create-application-app-1.png>)

4. You will see a list of guides that can help you for setting up, then click "Next".
5. In your IDE, define a custom URI scheme that the users will be redirected back to your app after they have authenticated with Authgear, e.g. `com.myapp.example://host/path`.[^1]
6. Head back to Authgear Portal, fill in the Redirect URI that you have defined in the previous steps.

![](<../.gitbook/assets/edit-application-app.png>)

7. Click "Save" in the top tool bar and keep the **Client ID**. You can also obtain it again from the Applications list later.
8. (Optional) Click the item in the applications list if you wish to configure more authentication settings.

{% hint style="info" %}
If you wish to [validate JSON Web Token (JWT) in your own application server](../backend-integration/jwt), select "Issue JWT as access token".[^2] If you wish to [forward authentication requests to Authgear Resolver Endpoint](../backend-integration/nginx), leave this unchecked. See comparisons in [Backend Integration](../backend-integration/).
{% endhint %}

![](<../.gitbook/assets/application-jwt.png>)

{% endtab %}

{% tab title="authgear.yaml (self-deployed)" %}
```yaml
oauth:
  clients:
    - name: your_app_name
      client_id: a_random_generated_string
      redirect_uris:
        - "com.myapp.example://host/path"
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
import 'package:flutter_authgear/flutter_authgear.dart';

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

  Future<void> _init() async {
    _authgear = Authgear(endpoint: "ENDPOINT", clientID: "CLIENT_ID");
    await _authgear.configure();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "MyApp",
      home: Scaffold(
        appBar: AppBar(title: const Text("MyApp")),
        body: TextButton(
          child: Text("Authenticate"),
          onPressed: _onPressedAuthenticate,
        ),
      ),
    );
  }

  Future<void> _onPressedAuthenticate() async {
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

{% content-ref url="backend-integration/" %}
[backend-integration](backend-integration/)
{% endcontent-ref %}

## Flutter SDK Reference

For detailed documentation on the Flutter SDK, visit [Flutter SDK Reference](https://authgear.github.io/authgear-sdk-flutter/)

### Footnote
[^1]: For futher instruction on setting up custom URI scheme in Flutter, see [https://docs.flutter.dev/development/ui/navigation/deep-linking](https://docs.flutter.dev/development/ui/navigation/deep-linking)
[^2]: For more explaination on JWT, see [https://en.wikipedia.org/wiki/JSON_Web_Token](https://en.wikipedia.org/wiki/JSON_Web_Token)