---
description: Authgear is an authentication server for web and mobile apps.
---

# Authgear Overview

**Authgear** is an authentication server for web and mobile apps. It is highly customizable yet comes with sensible defaults. It is built on top of [OpenID Connect (OIDC) standard](https://openid.net/connect/), making it very easy to integrate. It supports single sign-on (SSO) via integration with popular providers such as Google, Apple, and Azure Active Directory (AD). It also supports logging in via email address and phone number with one-time password or traditional password with strong password policy. Multiple factor authentication with time-based one-time password or out-out-band one-time password is available out of the box. Users can manage their sessions in the web-based settings page. Authgear, together with popular reverse proxies such as Nginx, can work as the authentication proxy for your other services.

### Concepts

Authgear consists of these major parts:

* **Authgear server**
  * It is an OpenID Connect (OIDC) compatible authentication service
  * It provides wide range of authentication and user management features
  * The [Authgear Resolver Endpoint](get-started/backend-integration/nginx.md) or [JWT access tokens](get-started/backend-integration/jwt.md) can be used to authenticate incoming requests
* **Authgear SDKs**
  * Used for developer to integrate Authgear into their apps. It can be used to perform user authentication, get the user profile, send authenticated requests
  * Explore: [Web (JavaScript)](get-started/website.md), [React Native](get-started/react-native.md), [iOS](get-started/ios.md), [Android](get-started/android/), [Flutter](get-started/flutter.md), [Xamarin](get-started/xamarin.md)
* **Admin API**
  * For backend servers to perform administrative tasks. Most things about user management you can do in the Authgear Portal, you can do it with [Admin API](apis/admin-api/)
* **Authgear Portal**
  * You can use the Authgear Portal for configuring your project, manage user, check [audit log](monitor/audit-log.md), customize the behavior by [event hooks](integrate/events-hooks/).
* **AuthUI**
  * The prebuilt UI for end-users to complete authentication and perform [account settings](integrate/auth-ui.md)
  * It can be [customized](customize/branding.md) to fit your company's branding
* **Events and Hooks**
  * Use [event and hooks](integrate/events-hooks/) to get the information about events that happened (non-blocking) and change the process of Authgear server (blocking)

### Quickstart guides

Adding Authgear to your single-page web app? Follow these tutorials to integrate Authgear with your favorite front-end frameworks.

* [React](tutorials/spa/react.md)
* [Vue](tutorials/spa/vue.md)
