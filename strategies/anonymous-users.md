---
description: >-
  Allow guest users to use your apps and website and promote to regular users
  later.
---

# Anonymous Users

## Overview

You can use the create an Anonymous User account for the guests in your apps, so they can carry out interactions just like a normal user. For example, guests can post comments and save preference in your social platform before setting email and password. The user session will persist even the app has been closed.

This improves the app experience because the user do not need to set up authenticators until further down the user journey, while still enjoying most of the app features. For app developers, the ability to create and assign Anonymous User also makes it easier to link the activities of an individual before and after sign-up.&#x20;

{% hint style="warning" %}
This page is WIP, sample code will be added.
{% endhint %}

## Enable Anonymous User in your application

1. In the **Portal** , go to the **Anonymous Users** page
2. Turn on the **Enable anonymous users?** toggle in the page and click **Save**

## Using the SDK

### Sign up as an Anonymous User

This will create an Anonymous User for the session. Subsequent requests from the end-user in the session can be identify by the same `sub`

{% tabs %}
{% tab title="React Native" %}

{% endtab %}

{% tab title="iOS" %}

{% endtab %}

{% tab title="Android" %}

{% endtab %}

{% tab title="Web" %}

{% endtab %}
{% endtabs %}

### Check the UserInfo object

After "signing up" as an anonymous user, you can [retrieve the "UserInfo" object](../integrate/user-profile.md#userinfo-endpoint) and see the `sub` of the end-user.

**UserInfo**

```json
{
  "sub": "...",
  "isVerified": false,
  "isAnonymous": true
}
```

### Promotion of an Anonymous User

`promoteAnonymousUser` function can be called to promote an anonymous user to a regular user with login ID (e.g. email, phone number) and authenticators (e.g. password). The end-user will be prompted a sign up page to complete the promotion. The `sub` of an end-user will remain the same after promotion.

{% tabs %}
{% tab title="React Native" %}

{% endtab %}

{% tab title="iOS" %}

{% endtab %}

{% tab title="Android" %}

{% endtab %}

{% tab title="Web" %}

{% endtab %}
{% endtabs %}

## User Lifetime

### Mobile apps

On Mobile SDKs, creating an anonymous user will create a key-pair. The key-pair is stored in the native encrypted store on the mobile device. The end-user can always re-login to the same anonymous user with the key-pair. Such anonymous user will become inaccessible when the encrypted store is removed.

### Web apps and websites

On the Web SDK, there will be no key-pair created. Therefore the end-user will not be able to login to the same Anonymous User after the their session become invalid. For cookie-based authentication, it is controlled by the "idle timeout" and "session lifetime" of the **Cookie**. For token-based authentication, it is controlled by the "idle timeout" and "token lifetime" of the **Refresh Token**.

In other words, The anonymous user account lifetime is the same as the logged in session lifetime.

To adjust the lifetime settings, change the timeouts and lifetimes in **Portal** > **Applications** accordingly.

#### Caution for high traffic websites

You should create anonymous users only when necessary in the user journey to prevent creating excessive orphan accounts in your tenant.
