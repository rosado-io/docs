---
description: >-
  Authgear provides a wide range of prebuilt frontend for the authentication
  related features of your apps
---

# User Settings

## Actions in the settings page

The end-user can perform the following actions on the setting page:

* Change their password.
* Add or change their email, phone number or username.
* Connect or disconnect to identity providers.
* Manage the signed in sessions.
* Enable or disable 2-step verification.
* and many more.

## Open the settings page directly in websites

In case your application is a website, the web SDK does not provide a method to open the settings page. However, you can construct the URL to the settings page as follows:

`https://<your-app-endpoint>/settings`

And then you can just set this as the `href` of your anchor tag

```html
<div>
    <a href="{SETTINGS_PAGE_URL}">Open Settings Page</a>
</div>
```

## Open the settings page with the SDK in mobile apps

If you are working on a mobile apps, you can open the settings page using the SDK. When the end-user has signed in, the SDK provides a method to open the settings page in a webview.

{% tabs %}
{% tab title="React Native" %}
```typescript
import React, { useCallback } from "react";
import authgear, { Page } from "@authgear/react-native";
import { View, Button } from "react-native";

function SettingsScreen() {
  const onPressOpenSettingsPage = useCallback(() => {
    authgear.open(Page.Settings).then(() => {
      // When the promise resolves, the webview have been closed.
    });
  }, []);
  return (
    <View>
      <Button
        title="Open Settings Page"
        onPress={onPressOpenSettingsPage}
      />
    </View>
  );
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
func onPressOpenSettingsPage(sender: UIButton, forEvent event: UIEvent) {
    authgear.open(.settings) {
        // When the completion handler is called, the webview is closed.
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
public void onClickOpenSettingsPage() {
    authgear.open(Page.Settings, null, new OnOpenURLListener() {
        @Override
        public void onClosed() {
            // The webview is closed.
        }

        @Override
        public void onFailed(Throwable throwable) {
            // Some error occured.
        }
    });
}
```
{% endtab %}
{% endtabs %}

## Back to my app button

In web-based application, you may want to add the "Back to my app" button to the settings page so the user can navigate back to your website after changing the settings.

1. Go to **Portal** > **UI Settings**
2. Provide the URL in **Back to Your App Link** and click **Save**
