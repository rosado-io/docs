---
description: Add login to your native mobile application
---

# Native mobile app \(iOS, Android or React Native\)

By using Authgear, you can add a login to your native mobile application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

## **Overview**

### **How it works**

Your app server will receive a request with the access token

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgcGFydGljaXBhbnQgQ2xpZW50QXBwIGFzIE1vYmlsZSBBcHBcbiAgICBwYXJ0aWNpcGFudCBBdXRoZ2VhciBhcyBBdXRoZ2VhclxuICAgIHBhcnRpY2lwYW50IEFwcEJhY2tlbmQgYXMgWW91ciBBcHAgU2VydmVyXG4gICAgQ2xpZW50QXBwLT4-QXV0aGdlYXI6IDEuIFVzZXIgYXV0aGVudGljYXRlcyB3aXRoIEF1dGhnZWFyXG4gICAgQXV0aGdlYXItPj5DbGllbnRBcHA6IDIuIEF1dGhnZWFyIHJlc3BvbmRzIHdpdGggdGhlIGFjY2VzcyB0b2tlbiBhbmQgcmVmcmVzaCB0b2tlblxuICAgIENsaWVudEFwcC0-PkFwcEJhY2tlbmQ6IDMuIFJlcXVlc3Qgd2l0aCBhY2Nlc3MgdG9rZW5cbiAgICBBcHBCYWNrZW5kLT4-QXBwQmFja2VuZDogNC4gVmVyaWZ5IFJlcXVlc3RcbiAgICBBcHBCYWNrZW5kLT4-Q2xpZW50QXBwOiA1LiBTZXJ2ZXIgcmVzcG9uZHMgd2l0aCB0aGUgcmVxdWVzdGVkIGluZm9ybWF0aW9uXG4gICAgICAgICAgICAiLCJtZXJtYWlkIjp7fSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)

### **Verify request in your app server**

To verify the request in your app server, you can choose **Forwarding authentication to Authgear Resolver Endpoint** or **Verify JSON Web Token \(JWT\) in your app server.**

![Forwarding authentication to Authgear Resolver Endpoint](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXV0aGdlYXJbQXV0aGdlYXJdXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHwgRm9yd2FyZCBhdXRoZW50aWNhdGlvbiB0byA8YnIvPiBBdXRoZ2VhciByZXNvbHZlciBlbmRwb2ludCB8IGF1dGhnZWFyXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

![Verify JSON Web Token \(JWT\) in your app server](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHxWZXJpZnkgSldUIHRva2VuIHwgYXBwXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

## Get Started <a id="get-started"></a>

The following tutorials show you how to add user login to your mobile app using Authgear.

### 1. Frontend Integration <a id="1-frontend-integration"></a>

Choose your platforms below:

{% page-ref page="../getting-started/react-native.md" %}

{% page-ref page="../getting-started/android/" %}

{% page-ref page="../getting-started/ios.md" %}

### 2. Backend Integration <a id="2-backend-integration"></a>

{% page-ref page="../getting-started/backend-integration/" %}

