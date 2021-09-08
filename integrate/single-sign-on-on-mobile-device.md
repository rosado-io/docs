# Single Sign-on on mobile device

On mobile device, Authgear uses a webview that can be configured to share cookie with the system browser. The system browser is Safari on iOS, while the system browser is Chrome on Android. If you have a mobile application and a website, you may wish to enable Single Sign-on mobile device so that

1. The end-user installed your app. They signed in in the app. Later on when they visit your website with the system browser, they are signed in.
2. The end-user have been using your website. One day they decided to install your app. When they are signing in in the app, they will see a continue screen so that they can continue with just a click, without authenticating themselves again.

You can turn on this feature when you configure the SDK.

{% tabs %}
{% tab title="React Native" %}
```typescript
authgear.configure({
    shareSessionWithSystemBrowser: true,
});
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
    null,
    true // shareSessionWithDeviceBrowser = true
);
```
{% endtab %}
{% endtabs %}
