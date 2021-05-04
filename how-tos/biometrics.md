# Biometrics login

## Overview

Authgear supports enabling biometrics login in native mobile application. To set this up, you will need to

1. Enable biometrics login in your application.
1. Sign up or login an user in your mobile application, use the mobile SDK to enable biometrics login.

## Enable biometrics login in your application

{% tabs %}
{% tab title="Portal" %}
1. In portal, go to "Biometric Authentication".
1. Turn on "Enable biometric authentication".
1. Click save.
{% endtab %}
{% tab title="authgear.yaml" %}
```yaml
authentication:
  identities:
    ...
    # Add biometric along with the other enabled identities
    - biometric
identity:
  biometric:
    # Enable listing biometrics login in user setting page, default false
    list_enabled: true
```
{% endtab %}
{% endtabs %}

## Enable biometrics login in mobile SDK

In the following section, we will show you how to use biometrics login in the SDK. In the SDK code snippet, `authgear` is referring to the configured Authgear container.

- Check if current device supports biometrics login before calling any biometric api.

{% tabs %}
{% tab title="iOS" %}
```swift
// check if current device supports biometrics login
var supported = false
do {
    try authgear.checkBiometricSupported()
    supported = true
} catch {}

if supported {
    // biometrics login is supported
}
```
{% endtab %}
{% tab title="Android" %}
```java
boolean supported = false;
try {
    // biometric login is supported SDK_INT >= 23 (Marshmallow)
    if (Build.VERSION.SDK_INT >= 23) {
        // check if current device supports biometrics login
        authgear.checkBiometricSupported(
                this.getApplication(),
                ALLOWED
        );
        supported = true;
    }
} catch (Exception e) {}
if (supported) {
    // biometrics login is supported
}
```
{% endtab %}
{% tab title="React Native" %}
```javascript
// We will need the options for the other biometric api
const biometricOptions = {
  ios: {
    localizedReason: 'Use biometric to authenticate',
    constraint: 'biometryCurrentSet' as const,
  },
  android: {
    title: 'Biometric Authentication',
    subtitle: 'Biometric authentication',
    description: 'Use biometric to authenticate',
    negativeButtonText: 'Cancel',
    constraint: ['BIOMETRIC_STRONG' as const],
    invalidatedByBiometricEnrollment: true,
  },
};
// check if current device supports biometrics login
authgear
    .checkBiometricSupported(biometricOptions)
    .then(() => {
        // biometrics login is supported
    })
    .catch(() => {
        // biometrics login is not supported
    });
```
{% endtab %}
{% endtabs %}

- Enable biometrics login for logged in user

{% tabs %}
{% tab title="iOS" %}
```swift
// provide localizedReason for requesting authentication
// which displays in the authentication dialog presented to the user
authgear.enableBiometric(
    localizedReason: "REPLACE_WITH_LOCALIZED_REASON",
    constraint: .biometryCurrentSet
) { result in
    if case let .failure(error) = result {
        // failed to enable biometric with error
    } else {
        // enabled biometric successfully
    }
}
```
{% endtab %}
{% tab title="Android" %}
```java
// We will need the options for the other biometric api
BiometricOptions biometricOptions = new BiometricOptions(
    activity, // FragmentActivity
    "Biometric authentication", // title
    "Biometric authentication", // subtitle
    "Use biometric to authenticate", // description
    "Cancel", // negativeButtonText
    ALLOWED, // allowedAuthenticators
    true // invalidatedByBiometricEnrollment
);
authgear.enableBiometric(
    biometricOptions,
    new OnEnableBiometricListener() {
        @Override
        public void onEnabled() {
            // enabled biometrics login successfully
        }

        @Override
        public void onFailed(Throwable throwable) {
            // failed to enable biometric with error
        }
    }
);
```
{% endtab %}
{% tab title="React Native" %}
```javascript
authgear
    .enableBiometric(biometricOptions)
    .then(() => {
        // enabled biometrics login successfully
    })
    .catch((err) => {
        // failed to enable biometric with error
    });
```
{% endtab %}
{% endtabs %}

- Check if current device enabled biometrics login, we should check this before asking user to login with biometrics credentials

{% tabs %}
{% tab title="iOS" %}
```swift
var enabled = (try? authgear.isBiometricEnabled()) ?? false
```
{% endtab %}
{% tab title="Android" %}
```java
boolean enabled = false;
try {
    enabled = authgear.isBiometricEnabled();
} catch (Exception e) {}
```
{% endtab %}
{% tab title="React Native" %}
```javascript
authgear
    .isBiometricEnabled()
    .then((enabled) => {
        // show if biometrics login is enabled
    })
    .catch(() => {
        // failed to check the enabled status
    });
```
{% endtab %}
{% endtabs %}

- Login with biometrics credentials

{% tabs %}
{% tab title="iOS" %}
```swift
authgear.authenticateBiometric { result in
    switch result {
        case let .success(authResult):
            let userInfo = authResult.userInfo
            // logged in successfully
        case let .failure(error):
            // failed to login
        }
}
```
{% endtab %}
{% tab title="Android" %}
```java
authgear.authenticateBiometric(
    biometricOptions,
    new OnAuthenticateBiometricListener() {
        @Override
        public void onAuthenticated(UserInfo userInfo) {
            // logged in successfully
        }

        @Override
        public void onAuthenticationFailed(Throwable throwable) {
            // failed to login
        }
    }
);
```
{% endtab %}
{% tab title="React Native" %}
```javascript
authgear
    .authenticateBiometric(biometricOptions)
    .then(({userInfo}) => {
        // logged in successfully
    })
    .catch((e) => {
        // failed to login
    });
```
{% endtab %}
{% endtabs %}

- Disable biometrics login in the current device

{% tabs %}
{% tab title="iOS" %}
```swift
do {
    try authgear.disableBiometric()
    // disabled biometrics login successfully
} catch {
    // failed to disable biometrics login
}
```
{% endtab %}
{% tab title="Android" %}
```java
try {
    authgear.disableBiometric();
    // disabled biometrics login successfully
} catch (Exception e) {
    // failed to disable biometrics login
}
```
{% endtab %}
{% tab title="React Native" %}
```javascript
authgear
    .disableBiometric()
    .then(() => {
        // disabled biometrics login successfully
    })
    .catch((err) => {
        // failed to disable biometrics login
    });
```
{% endtab %}
{% endtabs %}
