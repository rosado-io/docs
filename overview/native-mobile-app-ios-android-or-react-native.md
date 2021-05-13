---
description: Add login to your native mobile application
---

# Native mobile app \(iOS, Android or React Native\)

By using Authgear, you can add a login to your native mobile application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

### Overview of application flow

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IE1vYmlsZUFwcCBhcyBNb2JpbGUgQXBwXG4gIHBhcnRpY2lwYW50IEFwcEJhY2tlbmQgYXMgWW91ciBhcHAgc2VydmVyXG4gIHBhcnRpY2lwYW50IEF1dGhnZWFyIGFzIEF1dGhnZXJcbiAgcmVjdCByZ2IoMjMwLCAyMzYsIDI0MSlcbiAgTm90ZSByaWdodCBvZiBNb2JpbGVBcHA6IFRyaWdnZXIgdXNlciBsb2dpbiBcbiAgTW9iaWxlQXBwLT4-QXV0aGdlYXI6IFVzaW5nIEF1dGhnZWFyIFNESyB0byB0cmlnZ2VyIGxvZ2luIGZsb3cgPGJyLz4oQXV0aG9yaXphdGlvbiBDb2RlIEZsb3cpXG4gIEF1dGhnZWFyLT4-TW9iaWxlQXBwOiBVc2VyIGF1dGhlbnRpY2F0ZXMgd2l0aCBBdXRoZ2VhclxuICBlbmRcbiAgcmVjdCByZ2IoMjMwLCAyMzYsIDI0MSlcbiAgTm90ZSByaWdodCBvZiBNb2JpbGVBcHA6IE1ha2luZyBhcHBsaWNhdGlvbiByZXF1ZXN0XG4gIE1vYmlsZUFwcC0-PkFwcEJhY2tlbmQ6IFNlbmRpbmcgYXBwbGljYXRpb24gcmVxdWVzdCB3aXRoIGFjY2VzcyB0b2tlbiA8YnIvPiBieSBoZWxwIG9mIEF1dGhnZWFyIFNES1xuICBBcHBCYWNrZW5kLT4-QXV0aGdlYXI6IFNldHVwIHByb3h5IHRvIGludGVncmF0ZSB3aXRoIEF1dGhnZWFyIHJlc29sdmVyIHRvIGF1dGhlbnRpY2F0ZSBIVFRQIHJlcXVlc3QgPGJyLz4gZS5nLiBuZ2lueCBuZ3hfaHR0cF9hdXRoX3JlcXVlc3RfbW9kdWxlXG4gIEF1dGhnZWFyLT4-QXBwQmFja2VuZDogQXV0aGdlYXIgcmVwb25zZSBIVFRQIGhlYWRlcnMgd2l0aCB1c2VyIGxvZ2luIHN0YXRlPGJyLz5wcm94eSB1cGRhdGUgcmVxdWVzdCB3aXRoIGhlYWRlcnMgPGJyLz4gYXBwbGljYXRpb24gcmVhZCB0aGUgaGVhZGVycyBmb3IgdXNlciBsb2dpbiBzdGF0ZVxuICBBcHBCYWNrZW5kLT4-TW9iaWxlQXBwOiBSZXNwb25zZSB0byBtb2JpbGUgYXBwXG4gIGVuZCIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0Iiwic2VxdWVuY2UiOnsic2hvd1NlcXVlbmNlTnVtYmVycyI6dHJ1ZX19LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

1. When the user clicks login in your application, Authgear SDK redirects the user to the Authgear server which opens ASWebAuthenticationSession in iOS or a Custom Chrome Tab in Android.
2. User authenticates with Authgear with your configured login methods.
3. Sending application request with access token with the help of Authgear SDK
4. Set up a reverse proxy in your infrastructure to authenticate HTTP requests, it can be any application proxy that supports auth requests. The proxy will request the Authgear resolver.
5. Authgear resolves parses the access token and returns HTTP headers with include user login state, the proxy will update the HTTP request with those headers. And you can read those headers in your application code to know who is the currently logged user.
6. Your app server returns a response to the mobile app. e.g. Returns user's content or HTTP 401 if the user is not logged in.

### Setup Overview

1. Setup Authgear and integrate your application with Authgear SDK
   1. [React-Native](../getting-started/react-native.md)
   2. iOS
   3. Android
2. [Integrate your backend with Authgear to authenticate the request](../getting-started/backend-integration/)
3. [Using SDK to call your application server](../getting-started/using-sdk-to-call-your-application-server.md)

