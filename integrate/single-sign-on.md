---
description: >-
  Provide a seamless user experience across multiple products with the single
  sign-on feature.
---

# Single Sign-on on mobile devices

On mobile devices, Authgear uses a webview that can be configured to share the cookie with the system browser. The system browser is Safari on iOS, while the system browser is Chrome on Android. If you have both a mobile application and a website, you can enable the Single Sign-on feature so that

1. **From Apps to Website:** The end-user installs your app and signs in. Later on, when they visit your website with the system browser, they will be already signed in.
2. **From Website to Apps:** The end-user has been using your website. One day they decided to install your app. When they log in to the app, they will see a **continue screen** so that they can log in with just a click, without authenticating themselves again.

You can turn on this feature when you configure the SDK by setting the `shareSessionWithSystemBrowser` option to `true`.

{% tabs %}
{% tab title="React Native" %}
```typescript
authgear.configure({
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    shareSessionWithSystemBrowser: true,
});
```
{% endtab %}

{% tab title="Flutter" %}
```dart
final authgear = Authgear(
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    shareSessionWithSystemBrowser: true,
);
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
var authgearOptions = new AuthgearOptions
{
    ClientId = CLIENT_ID,
    AuthgearEndpoint = ENDPOINT,
    ShareSessionWithSystemBrowser = true,
};
// Android
#if __ANDROID__
var authgear = new AuthgearSdk(GetActivity().ApplicationContext, authgearOptions);
#else
#if __IOS__
var authgear = new AuthgearSdk(UIKit.UIApplication.SharedApplication, authgearOptions);
#endif
#endif
```
{% endtab %}

{% tab title="iOS" %}
```swift
Authgear(
    clientId: CLIENT_ID,
    endpoint: ENDPOINT,
    shareSessionWithSystemBrowser: true,
)
```
{% endtab %}

{% tab title="Android" %}
```kotlin
new Authgear(
    application,
    CLIENT_ID,
    ENDPOINT,
    null, // tokenStorage = default
    true // shareSessionWithSystemBrowser = true
);
```
{% endtab %}
{% endtabs %}

