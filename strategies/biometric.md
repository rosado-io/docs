# Biometric login

## Overview

Authgear supports enabling biometric login in the native mobile application. To set this up, you will need to

1. Enable biometric login in your application.
2. Sign up or log in a user in your mobile application, use the mobile SDK to enable biometric login.

## Enable biometric login in your application

{% tabs %}
{% tab title="Portal" %}
1. In the portal, go to "Biometric Authentication".
2. Turn on "Enable biometric authentication".
3. Click "Save".
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
    # Enable listing biometric login in user setting page, default false
    list_enabled: true
```
{% endtab %}
{% endtabs %}

## Enable biometric login in mobile SDK

In the following section, we will show you how to use biometric login in the SDK. In the SDK code snippet, `authgear` is referring to the configured Authgear container.

* Check if the current device supports biometric login before calling any biometric API.

{% tabs %}
{% tab title="iOS" %}
```swift
// check if current device supports biometric login
var supported = false
do {
    try authgear.checkBiometricSupported()
    supported = true
} catch {}

if supported {
    // biometric login is supported
}
```
{% endtab %}

{% tab title="Android" %}
```java
boolean supported = false;
try {
    // biometric login is supported SDK_INT >= 23 (Marshmallow)
    if (Build.VERSION.SDK_INT >= 23) {
        // check if current device supports biometric login
        authgear.checkBiometricSupported(
                this.getApplication(),
                ALLOWED
        );
        supported = true;
    }
} catch (Exception e) {}
if (supported) {
    // biometric login is supported
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
// check if current device supports biometric login
authgear
    .checkBiometricSupported(biometricOptions)
    .then(() => {
        // biometric login is supported
    })
    .catch(() => {
        // biometric login is not supported
    });
```
{% endtab %}

{% tab title="Flutter" %}
```dart
// We will need the options for the other biometric api
final ios = BiometricOptionsIOS(
    localizedReason: "Use biometric to authenticate",
    constraint: BiometricAccessConstraintIOS.biometryAny,
);
final android = BiometricOptionsAndroid(
    title: "Biometric Authentication",
    subtitle: "Biometric authentication",
    description: "Use biometric to authenticate",
    negativeButtonText: "Cancel",
    constraint: [BiometricAccessConstraintAndroid.biometricStrong],
    invalidatedByBiometricEnrollment: false,
);

try {
    // check if current device supports biometric login
    await authgear.checkBiometricSupported(ios: ios, android: android);
    // biometric login is supported
} catch (e) {
    // biometric login is not supported
}
```
{% endtab %}

{% endtabs %}

* Enable biometric login for logged in user

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
            // enabled biometric login successfully
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
        // enabled biometric login successfully
    })
    .catch((err) => {
        // failed to enable biometric with error
    });
```
{% endtab %}

{% tab title="Flutter" %}
```dart
try {
    await authgear.enableBiometric(ios: ios, android: android);
    // enabled biometric login successfully
} catch (e) {
    // failed to enable biometric with error
}
```
{% endtab %}
{% endtabs %}

* Check if the current device enabled biometric login, we should check this before asking the user to log in with biometric credentials

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
        // show if biometric login is enabled
    })
    .catch(() => {
        // failed to check the enabled status
    });
```
{% endtab %}

{% tab title="Flutter" %}
```dart
try {
    final enabled = await authgear.isBiometricEnabled();
    // show if biometric login is enabled
} catch (e) {
    // failed to check the enabled status
}
```
{% endtab %}
{% endtabs %}

* Login with biometric credentials

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

{% tab title="Flutter" %}
```dart
try {
    final userInfo = await authgear.authenticateBiometric(ios: ios, android: android);
    // logged in successfully
} catch (e) {
    // failed to login
}
```
{% endtab %}
{% endtabs %}

* Disable biometric login in the current device

{% tabs %}
{% tab title="iOS" %}
```swift
do {
    try authgear.disableBiometric()
    // disabled biometric login successfully
} catch {
    // failed to disable biometric login
}
```
{% endtab %}

{% tab title="Android" %}
```java
try {
    authgear.disableBiometric();
    // disabled biometric login successfully
} catch (Exception e) {
    // failed to disable biometric login
}
```
{% endtab %}

{% tab title="React Native" %}
```javascript
authgear
    .disableBiometric()
    .then(() => {
        // disabled biometric login successfully
    })
    .catch((err) => {
        // failed to disable biometric login
    });
```
{% endtab %}

{% tab title="Flutter" %}
```dart
try {
    await authgear.disableBiometric();
    // disabled biometric login successfully
} catch (e) {
    // failed to disable biometric login
}
```
{% endtab %}
{% endtabs %}

* Error handling

{% tabs %}
{% tab title="iOS" %}
```swift
if let authgearError = error as? AuthgearError {
    switch authgearError {
    case .cancel:
        // user cancel
    case .biometricPrivateKeyNotFound:
        // biometric info has changed. e.g. Touch ID or Face ID has changed.
        // user have to set up biometric authentication again
    case .biometricNotSupportedOrPermissionDenied:
        // user has denied the permission of using Face ID
    case .biometricNoPasscode:
        // device does not have passcode set up
    case .biometricNoEnrollment:
        // device does not have Face ID or Touch ID set up
    case .biometricLockout:
        // the biometric is locked out due to too many failed attempts
    default:
        // other error
        // you may consider showing a generic error message to the user
    }
}
```
{% endtab %}

{% tab title="Android" %}
```java
import com.oursky.authgear.BiometricLockoutException;
import com.oursky.authgear.BiometricNoEnrollmentException;
import com.oursky.authgear.BiometricNoPasscodeException;
import com.oursky.authgear.BiometricNotSupportedOrPermissionDeniedException;
import com.oursky.authgear.BiometricPrivateKeyNotFoundException;
import com.oursky.authgear.CancelException;


if (e instanceof CancelException) {
    // user cancel
} else if (e instanceof BiometricPrivateKeyNotFoundException) {
    // biometric info has changed
    // user have to set up biometric authentication again
} else if (e instanceof BiometricNoEnrollmentException) {
    // device does not have biometric set up
} else if (e instanceof BiometricNotSupportedOrPermissionDeniedException) {
    // biometric is not supported in the current device
    // or user has denied the permission of using biometric
} else if (e instanceof BiometricNoPasscodeException) {
    // device does not have unlock credential set up
} else if (e instanceof BiometricLockoutException) {
    // the biometric is locked out due to too many failed attempts
} else {
    // other error
    // you may consider showing a generic error message to the user
}
```
{% endtab %}

{% tab title="React Native" %}
```javascript
import {
    CancelError,
    BiometricPrivateKeyNotFoundError,
    BiometricNotSupportedOrPermissionDeniedError,
    BiometricNoEnrollmentError,
    BiometricNoPasscodeError,
    BiometricLockoutError,
} from '@authgear/react-native'

if (e instanceof CancelError) {
    // user cancel
} else if (e instanceof BiometricPrivateKeyNotFoundError) {
    // biometric info has changed. e.g. Touch ID or Face ID has changed.
    // user have to set up biometric authentication again
} else if (e instanceof BiometricNoEnrollmentError) {
    // device does not have biometric set up
    // e.g. have not set up Face ID or Touch ID in the device
} else if (e instanceof BiometricNotSupportedOrPermissionDeniedError) {
    // biometric is not supported in the current device
    // or user has denied the permission of using Face ID
} else if (e instanceof BiometricNoPasscodeError) {
    // device does not have unlock credential or passcode set up
} else if (e instanceof BiometricLockoutError) {
    // the biometric is locked out due to too many failed attempts
} else {
    // other error
    // you may consider showing a generic error message to the user
}
```
{% endtab %}

{% tab title="Flutter" %}
```dart
try {
    // ...
} on CancelException catch (e) {
    // user cancel
} on BiometricPrivateKeyNotFoundException catch (e) {
    // biometric info has changed. e.g. Touch ID or Face ID has changed.
    // user have to set up biometric authentication again
} on BiometricNoEnrollmentException catch (e) {
    // device does not have biometric set up
    // e.g. have not set up Face ID or Touch ID in the device
} on BiometricNotSupportedOrPermissionDeniedException catch (e) {
    // biometric is not supported in the current device
    // or user has denied the permission of using Face ID
} on BiometricNoPasscodeException catch (e) {
    // device does not have unlock credential or passcode set up
} on BiometricLockoutException catch (e) {
    // the biometric is locked out due to too many failed attempts
} catch (e) {
    // other error
    // you may consider showing a generic error message to the user
}
```
{% endtab %}

{% endtabs %}

