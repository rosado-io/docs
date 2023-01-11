---
description: >-
  Provide a seamless user experience across multiple products with the single
  sign-on feature.
---

# Single Sign-on

Single sign-on (SSO) is defined as login once, logged in all apps. If you have multiple mobile apps or websites that use the same Authgear project. You can configure your apps to turn on the SSO feature, so the end-users only have to enter their authentication credentials once.

If you are building cookie-based websites with the same root domain (e.g. `app1.example.com` / `app2.example.com`), you can skip this section. Sessions are shared among `*.example.com` automatically, see [detail](../get-started/authentication-approach/cookie-based.md).

If you are building token-based websites or mobile apps, you can enable the SSO feature via the SDK.

When SSO-enabled is ON, the end-user will need to enter their authentication credentials when they login to the first app. Later on, when they login to the second app, they will see a **continue screen** so that they can log in with just a click, without authenticating themselves again.

{% hint style="info" %}
It is important that when the SSO feature is ON, don't set the `prompt` parameter when authenticating (e.g. `prompt=login`). Otherwise, the end-user will need to login again.
{% endhint %}

When the end-user logout the SSO-enabled app, all the apps will be logged out at the same time.

You can turn on this feature when you configure the SDK by setting the **is sso enabled** option to `true`.

{% tabs %}
{% tab title="Web" %}
```typescript
authgear.configure({
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    sessionType: "refresh_token",
    isSSOEnabled: true,
});
```
{% endtab %}
{% tab title="React Native" %}
```typescript
authgear.configure({
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    isSSOEnabled: true,
});
```
{% endtab %}

{% tab title="Flutter" %}
```dart
final authgear = Authgear(
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    isSsoEnabled: true,
);
```
{% endtab %}

{% tab title="Xamarin" %}
```csharp
var authgearOptions = new AuthgearOptions
{
    ClientId = CLIENT_ID,
    AuthgearEndpoint = ENDPOINT,
    IsSsoEnabled = true,
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
    isSSOEnabled: true,
)
```
{% endtab %}

{% tab title="Android" %}
```java
new Authgear(
    getApplication(),
    CLIENT_ID,
    ENDPOINT,
    new PersistentTokenStorage(getApplication()),
    true // isSsoEnabled = true
);
```
{% endtab %}
{% endtabs %}

