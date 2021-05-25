---
description: Add login to your single page application
---

# Single-page app with access token

By using Authgear, you can add a login to your single-page application easily. Authgear supports various authentication methods, that you can easily turn on and configure in the portal.

## **Overview**

### **How it works**

Your app server will receive a request with the access token

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgcGFydGljaXBhbnQgQ2xpZW50QXBwIGFzIFNpbmdsZS1QYWdlIEFwcFxuICAgIHBhcnRpY2lwYW50IEF1dGhnZWFyIGFzIEF1dGhnZWFyXG4gICAgcGFydGljaXBhbnQgQXBwQmFja2VuZCBhcyBZb3VyIEFwcCBTZXJ2ZXJcbiAgICBDbGllbnRBcHAtPj5BdXRoZ2VhcjogMS4gVXNlciBhdXRoZW50aWNhdGVzIHdpdGggQXV0aGdlYXJcbiAgICBBdXRoZ2Vhci0-PkNsaWVudEFwcDogMi4gQXV0aGdlYXIgcmVzcG9uZHMgd2l0aCB0aGUgYWNjZXNzIHRva2VuIGFuZCByZWZyZXNoIHRva2VuXG4gICAgQ2xpZW50QXBwLT4-QXBwQmFja2VuZDogMy4gUmVxdWVzdCB3aXRoIGFjY2VzcyB0b2tlblxuICAgIEFwcEJhY2tlbmQtPj5BcHBCYWNrZW5kOiA0LiBWZXJpZnkgUmVxdWVzdFxuICAgIEFwcEJhY2tlbmQtPj5DbGllbnRBcHA6IDUuIFNlcnZlciByZXNwb25kcyB3aXRoIHRoZSByZXF1ZXN0ZWQgaW5mb3JtYXRpb25cbiAgICAgICAgICAgICIsIm1lcm1haWQiOnt9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

### **Verify request in your app server**

To verify the request in your app server, you can choose **Forwarding authentication to Authgear Resolver Endpoint** or **Verify JSON Web Token \(JWT\) in your app server.**

![Forwarding authentication to Authgear Resolver Endpoint](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXV0aGdlYXJbQXV0aGdlYXJdXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHwgRm9yd2FyZCBhdXRoZW50aWNhdGlvbiB0byA8YnIvPiBBdXRoZ2VhciByZXNvbHZlciBlbmRwb2ludCB8IGF1dGhnZWFyXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

![Verify JSON Web Token \(JWT\) in your app server](https://mermaid.ink/img/eyJjb2RlIjoiZmxvd2NoYXJ0IFREXG4gICAgYXBwW1lvdXIgQXBwIFNlcnZlcl1cbiAgICBcbiAgICBhcHAgLS0-IHxWZXJpZnkgSldUIHRva2VuIHwgYXBwXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

##  <a id="get-started"></a>

## Get Started

The following tutorials show you how to add user login to your single app using Authgear.

### 1. Frontend Integration

{% page-ref page="../getting-started/website.md" %}

### 2. Backend Integration

{% page-ref page="../getting-started/backend-integration/" %}

