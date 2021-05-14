---
description: Add login to your single page application
---

# Single-page app with access token

By using Authgear, you can add a login to your single-page application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

Authgear also helps you to protect your server from unauthorized access.

### How it works

![](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgY2xpZW50W1NpbmdsZS1QYWdlIEFwcF1cbiAgICBhdXRoZ2VhcltBdXRoZ2Vhcl1cbiAgICBhcHBbWW91ciBBcHAgU2VydmVyXVxuICAgIFxuICAgIGNsaWVudCAtLT4gfDEuIFVzZXIgYXV0aGVuY2lhdGUgPGJyLz4gd2l0aCBBdXRoZ2VhcnwgYXV0aGdlYXJcbiAgICBjbGllbnQgLS0-IHwyLiBSZXF1ZXN0IHlvdXIgYXBwIHNlcnZlcnwgYXBwXG4gICAgYXBwIC0tPiB8My4gT3B0aW9uIDE6IFJldmVyc2UgcHJveHkgZGVsZWdhdGVzIDxici8-IGF1dGhlbnRpY2F0aW9uIHRvIEF1dGhnZWFyIHJlc29sdmVyIHwgYXV0aGdlYXJcbiAgICBhcHAgLS0-IHwzLiBPcHRpb24gMjogVmVyaWZ5IEpXVCB0b2tlbiB8IGFwcFxuIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)

1. User authenticates with Authgear and Authgear responds with refresh token and access token.
2. The app calls your web server with the Authgear access token.
3. In your backend, check if the request is authenticated by setting up Auth forward in your reverse proxy \(e.g. Nginx\) or verify JSON Web Token \(JWT\) in your app server.

### Setup Overview

1. [Setup Authgear and adding login with Authgear JS SDK](../getting-started/website.md)
2. [Integrate your backend with Authgear to authenticate the request](../getting-started/backend-integration/)
3. [Using SDK to call your application server](../getting-started/using-sdk-to-call-your-application-server.md)





