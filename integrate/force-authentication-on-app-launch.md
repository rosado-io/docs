# Force authentication on app launch

If your mobile app has some security requirements similar to that of banking mobile applications, you wish your end-users to authenticate themselves every time they use your app.

By default, the SDK stores the refresh token in a persistent storage specific to your app. Your app works like social apps like Facebook, the end-user signed in once and the session lasts for a long period.

You can alter this behavior by switching to a transient storage.

{% tabs %}
{% tab title="React Native" %}
```typescript
import authgear, { TransientTokenStorage } from "@authgear/react-native";

authgear.configure({
    clientID: CLIENT_ID,
    endpoint: ENDPOINT,
    tokenStorage: new TransientTokenStorage(),
});
```
{% endtab %}

{% tab title="iOS" %}
```swift
Authgear(
    clientId: CLIENT_ID,
    endpoint: ENDPOINT,
    tokenStorage: TransientTokenStorage()
)
```
{% endtab %}

{% tab title="Android" %}
```java
new Authgear(
    application,
    CLIENT_ID,
    ENDPOINT,
    new TransientTokenStorage()
);
```
{% endtab %}
{% endtabs %}
