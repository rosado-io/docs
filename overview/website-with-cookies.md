---
description: Add login to your website
---

# Website with cookies \(e.g. Server-side rendered app, regular website, single-page app\)

By using Authgear, you can add a login to your website easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

### Prerequisites

To authenticate with cookies, you will need to set up a custom domain for Authgear, so that your website and Authgear are under the same root domains. e.g. Your website is `yourdomain.com`, and Authgear with a custom domain `auth.yourdomain.com`. 

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookies automatically.

### Overview of application flow

![](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgY2xpZW50W1VzZXIgQnJvd2VyXVxuICAgIGF1dGhnZWFyW0F1dGhnZWFyXVxuICAgIGFwcFtZb3VyIFdlYiBTZXJ2ZXJdXG4gICAgXG4gICAgY2xpZW50IC0tPiB8MS4gVXNlciBhdXRoZW5jaWF0ZSA8YnIvPiB3aXRoIEF1dGhnZWFyfCBhdXRoZ2VhclxuICAgIGNsaWVudCAtLT4gfDIuIFJlcXVlc3QgeW91ciBhcHBsaWNhdGlvbiBzZXJ2ZXJ8IGFwcFxuICAgIGFwcCAtLT4gfDMuIFJldmVyc2UgcHJveHkgZGVsZWdhdGVzIDxici8-IGF1dGhlbnRpY2F0aW9uIHRvIEF1dGhnZWFyIHJlc29sdmVyICB8IGF1dGhnZWFyXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)



1. User authenticates with Authgear and Authgear responds and sets cookies to the root domain.
2. User requests your server when browses your website, cookies will be included automatically.
3. In your backend, check if the request is authenticated by setting up Auth forward in your reverse proxy \(e.g. Nginx\).

### Setup Overview

1. [Setup Authgear and integrate your application with Authgear JS SDK](../getting-started/website.md)
2. [Integrate your backend with Authgear to authenticate the request](../getting-started/backend-integration/)



