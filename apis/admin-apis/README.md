---
description: The Admin API allows your server to manage users via a GraphQL endpoint.
---

# Admin APIs

The Admin API allows your server to manage users via a GraphQL endpoint. You can list users, search users, view user details, and many more. In fact, the user management part of the portal is built with the Admin API.

## The Admin API GraphQL endpoint

The Admin API GraphQL endpoint is at `/_api/admin/graphql`. For example, if your app is `myapp` , then the endpoint is `https://myapp.authgearapps.com/_api/admin/graphql` .

### Accessing the Admin API GraphQL endpoint

Accessing the Admin API GraphQL endpoint requires your server to generate a valid JWT and include it as `Authorization` HTTP header.

See [authorization-and-security.md](authorization-and-security.md "mention")to learn how to access the Admin API securely.

## Trying out the Admin API GraphQL endpoint

{% hint style="danger" %}
The GraphiQL tool is NOT a sandbox environment and all changes will be made on real data. Use with care!
{% endhint %}

The above instruction is for server-side integration. If you want to explore what it can do, you can visit the GraphiQL tool.

* Go to **Settings** -> **Admin API**
* Click on the **GraphiQL tool** link

### Inspecting the GraphQL schema

In the GraphiQL tool, you can toggle the schema documentation by pressing the Docs button in the top right corner.
