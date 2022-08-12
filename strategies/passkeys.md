---
description: >-
  Passkeys give users a simple and secure way to sign in to your apps and
  websites across platforms without passwords.
---

# Passkeys

{% hint style="warning" %}
This is a future in the roadmap planed to be released by the end of August.
{% endhint %}

Passkeys replace password and other passwordless login methods. It is built on the WebAuthn standard (also known as FIDO Sign-in), which uses public key cryptography to authenticate the user. With 1 click, Authgear upgrades your app to support this cutting-edge auth technology.

#### Platform support and Multi-devices

The passkey standard is supported on the latest versions of Chrome, Safari and Firefox browsers. On iOS 16 and macOS 13 (Ventura), [Apple has added passkey support](https://developer.apple.com/passkeys/) to the iCloud Keychain service. A passkey is synchronized and relayed with an iCloud account and can be used across a user's devices. [Google](https://blog.google/technology/safety-security/one-step-closer-to-a-passwordless-future/) and [Microsoft](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/expansion-of-fido-standard-and-new-updates-for-microsoft/ba-p/3290633) have also announced the support in their future platforms.

User can login to their accounts using their biometrics easily. On Apple devices, Touch ID and Face ID authorize use of the passkey which then authenticate the user on the app or website.

#### Hardware security keys

Besides the built-in support of all major desktop and mobile platforms, passkeys can also be stored in hardware security keys such as [YubiKeys](https://www.yubico.com/blog/passkeys-and-the-future-of-modern-authentication/), which provide the highest security against attacks.

## Add Passkeys to your apps with Authgear

Authgear add passkey feature to your apps and websites instantly. To enable it:

1. In your project portal, go to "**Authentication > Authenticators**"
2. In the "**Primary Authenticator**" section, turn on the "**Login with Passkeys**" toggle.
3. Press "Save" and :tada: your app now support passkey login!

{% hint style="info" %}
It will take time for the passkey technology to be available on everyone devices. In the transition stage, it is recommended to enable "Password" or "Passwordless via Email/Phone" in your project so users with non-compatible devices can access your app.

If you want to use ONLY passkeys in your app, it's perfectly supported too! Just turn off all "Primary Authenticators" and leave "**Login with Passkeys**" on.
{% endhint %}

## Support on different platforms

See the list of Passkey support via Authgear on different platforms.

* macOS 12: Passkey is supported on major browsers. However the credentials are deleted when clearing browser data.
* iOS 15.5: Passkey is supported on Safari and stored locally on device. Credential will be deleted by "Settings > Safari > Clear History and Website Data"
* iOS 16 Beta 3: Passkey is synced with iCloud Keychain. The individual credentials can be viewed and managed in "Settings > Passwords"
* Android 12: Not supported
