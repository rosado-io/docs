---
description: Add login to your native mobile application
---

# Native mobile app \(iOS, Android or React Native\)

By using Authgear, you can add a login to your native mobile application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

## Overview of application flow

![](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgY2xpZW50W01vYmlsZSBBcHBdXG4gICAgYXV0aGdlYXJbQXV0aGdlYXJdXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBjbGllbnQgLS0-IHwxLiBVc2VyIGF1dGhlbmNpYXRlIDxici8-IHdpdGggQXV0aGdlYXJ8IGF1dGhnZWFyXG4gICAgY2xpZW50IC0tPiB8Mi4gUmVxdWVzdCB5b3VyIGFwcCBzZXJ2ZXJ8IGFwcFxuICAgIGFwcCAtLT4gfDMuIE9wdGlvbiAxOiBSZXZlcnNlIHByb3h5IGRlbGVnYXRlcyA8YnIvPiBhdXRoZW50aWNhdGlvbiB0byBBdXRoZ2VhciByZXNvbHZlciB8IGF1dGhnZWFyXG4gICAgYXBwIC0tPiB8My4gT3B0aW9uIDI6IFZlcmlmeSBKV1QgdG9rZW4gfCBhcHBcbiIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)

1. User authenticates with Authgear and Authgear responds with refresh token and access token.
2. The app calls your web server with the Authgear access token.
3. In your backend, check if the request is authenticated by setting up Auth forward in your reverse proxy \(e.g. Nginx\) or verify JSON Web Token \(JWT\) in your app server.

## Get Started <a id="get-started"></a>

The following tutorials show you how to add user login to your single app using Authgear.

### 1. Frontend Integration <a id="1-frontend-integration"></a>

Choose your platforms below:

{% page-ref page="native-mobile-app-ios-android-or-react-native.md" %}

{% page-ref page="../getting-started/android/" %}

{% page-ref page="../getting-started/ios.md" %}

### 2. Backend Integration <a id="2-backend-integration"></a>

{% page-ref page="../getting-started/backend-integration/" %}

