---
description: Add login to your website
---

# Website with cookies \(e.g. Server side rendered app, regular website, single page app\)

By using Authgear, you can add a login to your website easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

### Prerequisites

To authenticate with cookies, you will need to setup a custom domain for Authgear, so that your website and Authgear are under the same root domains. e.g. Your website is `yourdomain.com`, and Authgear with a custom domain `auth.yourdomain.com`. 

In this setting, if you have multiple applications under `yourdomain.com`, all applications would share the same session cookies automatically.

### Overview of application flow

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIHBhcnRpY2lwYW50IENsaWVudEFwcCBhcyBVc2VyIEJyb3dzZXJcbiAgcGFydGljaXBhbnQgQXBwQmFja2VuZCBhcyBZb3VyIFdlYnNpdGUgPGJyLz4gKHlvdXJkb21haW4uY29tKVxuICBwYXJ0aWNpcGFudCBBdXRoZ2VhciBhcyBBdXRoZ2VyIDxici8-IChhdXRoLnlvdXJkb21haW4uY29tKVxuICByZWN0IHJnYigyMzAsIDIzNiwgMjQxKVxuICBOb3RlIHJpZ2h0IG9mIENsaWVudEFwcDogVHJpZ2dlciB1c2VyIGxvZ2luIFxuICBDbGllbnRBcHAtPj5BdXRoZ2VhcjogVXNpbmcgQXV0aGdlYXIgU0RLIHRvIHRyaWdnZXIgbG9naW4gZmxvd1xuICBBdXRoZ2Vhci0-PkNsaWVudEFwcDogVXNlciBhdXRoZW50aWNhdGVzIHdpdGggQXV0aGdlYXIgYW5kIGNvb2tpZXMgYXJlIHNldFxuICBlbmRcbiAgcmVjdCByZ2IoMjMwLCAyMzYsIDI0MSlcbiAgTm90ZSByaWdodCBvZiBDbGllbnRBcHA6IE1ha2luZyBhcHBsaWNhdGlvbiByZXF1ZXN0XG4gIENsaWVudEFwcC0-PkFwcEJhY2tlbmQ6IFVzZXIgcmVxdWVzdCB0aGUgd2Vic2l0ZTxici8-Y29va2llcyBhcmUgaW5jbHVkZWQgYXV0b21hdGljYWxseSBieSBicm93c2VyPGJyLz51bmRlciB0aGUgc2FtZSByb290IGRvbWFpbnNcbiAgQXBwQmFja2VuZC0-PkF1dGhnZWFyOiBTZXR1cCBwcm94eSB0byBpbnRlZ3JhdGUgd2l0aCBBdXRoZ2VhciByZXNvbHZlciB0byBhdXRoZW50aWNhdGUgSFRUUCByZXF1ZXN0IDxici8-IGUuZy4gbmdpbnggbmd4X2h0dHBfYXV0aF9yZXF1ZXN0X21vZHVsZVxuICBBdXRoZ2Vhci0-PkFwcEJhY2tlbmQ6IEF1dGhnZWFyIHJlcG9uc2UgSFRUUCBoZWFkZXJzIHdpdGggdXNlciBsb2dpbiBzdGF0ZTxici8-cHJveHkgdXBkYXRlIHJlcXVlc3Qgd2l0aCBoZWFkZXJzIDxici8-IGFwcGxpY2F0aW9uIHJlYWQgdGhlIGhlYWRlcnMgZm9yIHVzZXIgbG9naW4gc3RhdGVcbiAgQXBwQmFja2VuZC0-PkNsaWVudEFwcDogUmVuZGVyIHdlYiBwYWdlIHRvIHVzZXJcbiAgZW5kIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQiLCJzZXF1ZW5jZSI6eyJzaG93U2VxdWVuY2VOdW1iZXJzIjp0cnVlfX0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)



1. When the user clicks login in your website, Authgear SDK redirects the user to Authgear authenticates page.
2. User authenticates with Authgear with your configured login methods, Authgear response with cookies.
3. When users visit any pages of your website, cookies are included in the requests due to the same root domains.
4. Set up a reverse proxy in your infrastructure to authenticate HTTP requests, it can be any application proxy that supports auth requests. The proxy will request the Authgear resolver.
5. Authgear resolves parses the cookies header and returns HTTP headers with include user login state, the proxy will update the HTTP request with those headers. And you can read those headers in your application code.
6. Render web page to your user. 

### Setup Overview

1. [Setup Authgear and integrate your application with Authgear JS SDK](../getting-started/website.md)
2. [Integrate your backend with Authgear to authenticate the request](../getting-started/backend-integration/)



