# Biometric login

## Overview

{% hint style="info" %}
Biometric login is supported for the following operating systems:
- iOS 11.3 or higher
- Android 6.0 (API 23) or higher
{% endhint %}

Authgear supports enabling biometric login in the native mobile application. You will need to

1. Enable biometric login in your application via the portal.
2. In the mobile app, use the mobile SDK to enable biometric login for your users.

A pair of cryptographic keys will be generated upon registering biometric login. The private key will be stored securely in the device (using Keystore in Android and Keychain in iOS), while the public key is stored in the Authgear server. To authenticate the user, fingerprint or face is presented to unlock the private key, and a digital signed message is sent to the server to proof the authenticity of the user.&#x20;

Sounds overwhelming? Authgear's magic handle all these for you. Follow this guide to enable biometric login in your app with a few lines of code.

## Enable biometric authentication for your project

{% tabs %}
{% tab title="Portal" %}
1. In the portal, go to **Authentication > Biometric**.
2. Turn on **Enable biometric authentication**.
3. **Save** the settings.
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

### Biometric options

In the SDKs, a set of biometric options is required to check the support or enable biometric on the device.&#x20;

#### iOS

There are two options on iOS:

* `localizedReason` is the message string the user will see when FaceID or TouchID is presented
* `constraint` is an enum that constraint the access of key stored under different conditions:
  * `biometryAny`: The key is still accessible by Touch ID if fingers are added or removed, or by Face ID if the user is re-enrolled
  * `BiometricCurrentSet`: The key is invalidated if fingers are added or removed for Touch ID, or if the user re-enrolls for Face ID

See reference in Apple Developers Doc on [biometryAny](https://developer.apple.com/documentation/security/secaccesscontrolcreateflags/2937191-biometryany) and [biometryCurrentSet](https://developer.apple.com/documentation/security/secaccesscontrolcreateflags/2937192-biometrycurrentset).

#### Android

There are 6 options on Android:

* `title` is the Title of the biometric dialog presented to the users
* `subtitle` is the subtitle of the biometric dialog presented to the users
* `description` is the description of the biometric dialog presented to the users
* `negativeButtonText` is what the dismiss button says in the biometric dialog
* `constraint` is an array defines the requirement of security level, which can be `BIOMETRIC_STRONG`, `BIOMETRIC_WEAK`, `DEVICE_CREDENTIAL`. See reference in Android developers documentation on [`BiometricManager.Authenticators`](https://developer.android.com/reference/android/hardware/biometrics/BiometricManager.Authenticators)``
* `invalidatedByBiometricEnrollment` is a boolean that controls if the key pair will be invalidated if a new biometric is enrolled, or when all existing biometrics are deleted. See reference in Android developers documentation on [`KeyGenParameterSpec.Builder`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder#setInvalidatedByBiometricEnrollment\(boolean\)).

### Code examples

#### Check support

Always check if the current device supports biometric login before calling any biometric API, including before enabling biometric login and before using biometric to login.&#x20;

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

{% tab title="Xamarin" %}
```csharp
// We will need the options for the other biometric api
var ios = new BiometricOptionsIos
{
    LocalizedReason = "Use biometric to authenticate",
    AccessConstraint = BiometricAccessConstraintIos.BiometricAny,
};
var android = new BiometricOptionsAndroid
{
    Title = "Biometric Authentication",
    Subtitle = "Biometric authentication",
    Description = "Use biometric to authenticate",
    NegativeButtonText = "Cancel",
    AccessConstraint = BiometricAccessConstraintAndroid.BiometricOnly,
    InvalidatedByBiometricEnrollment = false,
};
var biometricOptions = new BiometricOptions
{
    Ios = ios, 
    Android = android
};
try
{
    // check if current device supports biometric login
    authgear.EnsureBiometricIsSupported(biometricOptions);
    // biometric login is supported
}
catch
{
    // biometric login is not supported
}
```
{% endtab %}
{% endtabs %}

#### Enable biometric login

Enable biometric login for logged in user

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

{% tab title="Xamarin" %}
```csharp
try
{
    await authgear.EnableBiometricAsync(biometricOptions);
    // enabled biometric login successfully
}
catch
{
    // failed to enable biometric with error
}
```
{% endtab %}
{% endtabs %}

#### Check if biometric has been enabled before

Before asking the user to log in with biometric, Check if biometric login has been enabled on the current device. I.e. Is the key pair exist on the device (Keystore in Android and Keychain in iOS).

This method will still return true even if all the fingerprint and facial data has been removed from the device. Before this method, you should use the "checkBiometricSupported" to check if biometry is supported in the device level.

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

{% tab title="Xamarin" %}
```csharp
try
{
    var enabled = await authgear.GetIsBiometricEnabledAsync();
    // show if biometric login is enabled
}
catch
{
    // failed to check the enabled status
}
```
{% endtab %}
{% endtabs %}

#### Login with biometric credentials

If biometric is supported and enabled, you can use the Authenticate Biometric method to log the user in. If the key pair is invalidated due to changes in the biometry settings, e.g added fingerprint or re-enrolled face data, the `biometricPrivateKeyNotFound` will be thrown. You should handle the error by the Disable Biometric method, and ask the user to register biometric login again.

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

{% tab title="Xamrin" %}
```csharp
try
{
    var userInfo = await authgear.AuthenticateBiometricAsync(biometricOptions);
    // logged in successfully
}
catch
{
    // failed to login
}
```
{% endtab %}
{% endtabs %}

#### Disable biometric login on the current device

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

{% tab title="Xamarin" %}
```csharp
try
{
    await authgear.DisableBiometricAsync();
    // disabled biometric login successfully
}
catch
{
    // failed to disable biometric login
}
```
{% endtab %}
{% endtabs %}

#### Error handling

In all methods related to biometric, the SDK may throw the following errors that describe the status of the biometry enrollment or the key pair stored on the device.

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

{% tab title="Xamarin" %}
```csharp
try 
{
    // ...
}
catch (OperationCanceledException ex)
{
    // user cancel
}
catch (BiometricPrivateKeyNotFoundException ex)
{
    // biometric info has changed. e.g. Touch ID or Face ID has changed.
    // user have to set up biometric authentication again
}
catch (BiometricNoEnrollmentException ex)
{
    // device does not have biometric set up
    // e.g. have not set up Face ID or Touch ID in the device
}
catch (BiometricNotSupportedOrPermissionDeniedException ex)
{
    // biometric is not supported in the current device
    // or user has denied the permission of using Face ID
}
catch (BiometricNoPasscodeException ex)
{
    // device does not have unlock credential or passcode set up
}
catch (BiometricLockoutException ex)
{
    // the biometric is locked out due to too many failed attempts
}
catch
{
    // other error
    // you may consider showing a generic error message to the user
}
```
{% endtab %}
{% endtabs %}
