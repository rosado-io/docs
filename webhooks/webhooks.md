---
description: >-
  Authgear can deliver webhook events to your server when important events such
  as new user signup happen.
---

# Webhooks

Authgear use webhooks to notify your application server when events happens in your Authgear project. To use webhooks you will need to:

1. Create webhook handler endpoint in your application server.
2. Configure Authgear to send event to your webhook endpoint.

There are two kind of events, **Blocking** and **Non-blocking** events.

* Blocking event is triggered before the operation is performed. The operation can be aborted by your webhook endpoint response.
* Non-blocking event is triggered after the operation is performed.

## Configure Authgear to send event to your webhook endpoint

{% tabs %}
{% tab title="Portal" %}
1. In portal, go to **Settings** &gt; **Webhooks**.
2. Add your webhooks endpoint in **Blocking Events** and **Non-Blocking Events**, depending on which event you want to listen to.
3. Click **Save**.
{% endtab %}

{% tab title="authgear.yaml" %}
```yaml
hook:
  blocking_handlers:
    - event: "user.pre_create"
      url: 'https://myapp.com/check_user_create'
  non_blocking_handlers:
    # listen to all events and filter events by type in request
    - events: ["*"]
      url: 'https://myapp.com/all_events'
    - events: ["user.created"]
      url: 'https://myapp.com/sync_user_creation'
```
{% endtab %}
{% endtabs %}

## Webhook Delivery

Webhook handlers must be in **HTTPS**. This ensures integrity and confidentiality of the delivery.

The webhook handler endpoint must be an absolute URL. The webhook events are sent to the webhook handler endpoint via `POST` requests. The webhook handlers must return a status code within the `2xx` range. Other status code is considered as a failed delivery.

Each event can have multiple handlers. The order of delivery is unspecified for non-blocking event. Blocking events are delivered in the source order as in the configuration.

## Blocking Events

Blocking events are delivered to webhook handlers synchronously, right before committing changes to the database.

Webhook handler must respond with a JSON body to indicate whether the operation should continue.

To let the operation to proceed, respond with `is_allowed` being set to `true`.

```javascript
{
  "is_allowed": true
}
```

To forbid the operation, respond with `is_allowed` being set to `false` and a non-empty `title` and `reason`.

```javascript
{
  "is_allowed": false,
  "title": "any title",
  "reason": "any string"
}
```

If any handler fails the operation, the operation is halted. The `title` and `reason` will be shown to the end-user as an error message.

Each blocking event webhook handler must response within 5 seconds. And all blocking handlers in total should take less than 10 seconds to complete. Otherwise it would be considered as a failed delivery.

## Non-blocking Events

Non-blocking events are delivered to webhook handlers asynchronously after the operation is performed \(i.e. committed into the database\).

The webhook handlers must response a status code within the `2xx` range within 60 seconds. Otherwise it would be considered as a failed delivery.

The response body of non-blocking event webhook handler is ignored.

## Webhook Request Body

All webhook events have the following shape:

```javascript
{
  "id": "0E1E9537-DF4F-4AF6-8B48-3DB4574D4F24",
  "seq": 435,
  "type": "user.pre_create",
  "payload": { /* ... */ },
  "context": { /* ... */ }
}
```

* `id`: The ID of the event.
* `seq`: A monotonically increasing signed 64-bit integer.
* `type`: The type of the webhook event.
* `payload`: The payload of the webhook event, varies with type.
* `context`: The context of the webhook event.

### Webhook Event Context

* `timestamp`: signed 64-bit UNIX timestamp of when this event is generated. Retried deliveries do not affect this field.
* `user_id`: The ID of the user associated with the event. It may be absent. For example, the user has not authenticated yet.
* `preferred_languages`: User preferred languages which are inferred from the request. Return values of `ui_locales` query if it is provided in auth ui, otherwise return languages in `Accept-Language` request header.
* `language`: User locale which is derived based on user's preferred languages and app's languages config.
* `triggered_by`: Triggered by indicates who triggered the events, values can be `user` or `admin_api`. `user` means it is triggered by user in auth ui. `admin_api` means it is triggered by admin api or admin portal.

## Webhook Event List

#### Blocking Events

* [user.pre\_create](webhooks.md#userpre_create)

#### Non-blocking Events

* [user.created](webhooks.md#usercreated)
* [user.authenticated](webhooks.md#userauthenticated)
* [user.anonymous.promoted](webhooks.md#useranonymouspromoted)
* [identity.email.added](webhooks.md#identityemailadded)
* [identity.email.removed](webhooks.md#identityemailremoved)
* [identity.email.updated](webhooks.md#identityemailupdated)
* [identity.phone.added](webhooks.md#identityphoneadded)
* [identity.phone.removed](webhooks.md#identityphoneremoved)
* [identity.phone.updated](webhooks.md#identityphoneupdated)
* [identity.username.added](webhooks.md#identityusernameadded)
* [identity.username.removed](webhooks.md#identityusernameremoved)
* [identity.username.updated](webhooks.md#identityusernameupdated)
* [identity.oauth.connected](webhooks.md#identityoauthconnected)
* [identity.oauth.disconnected](webhooks.md#identityoauthdisconnected)

#### user.pre\_create

Occurs right before the user creation. User can be created by user signup, user signup as anonymous user, or created by the admin via the Portal or API. Operation can be aborted by providing specific response in your webhook, details see [Webhook Blocking Events](webhooks.md#blocking-events).

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identities": [ { /* ... */ } ]
  }
}
```

#### user.created

Occurs after a new user is created. User can be created by user signup, user signup as anonymous user, or created by the admin via the Portal or API.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identities": [ { /* ... */ } ]
  }
}
```

#### user.authenticated

Occurs after user logged in.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "session": { /* ... */ }
  }
}
```

#### user.anonymous.promoted

Occurs whenever an anonymous user is promoted to normal user.

```javascript
{
  "payload": {
    "anonymous_user": { /* ... */ },
    "user": { /* ... */ },
    "identities": [{ /* ... */ }]
  }
}
```

#### identity.email.added

Occurs when a new email is added to existing user. Email can be added by user in setting page, added by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.email.removed

Occurs when email is removed from existing user. Email can be removed by user in setting page, removed by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.email.updated

Occurs when email is updated. Email can be updated by user in setting page.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.phone.added

Occurs when a new phone number is added to existing user. Phone number can be added by user in setting page, added by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.phone.removed

Occurs when phone number is removed from existing user. Phone number can be removed by user in setting page, removed by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.phone.updated

Occurs when phone number is updated. Phone number can be updated by user in setting page.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.username.added

Occurs when a new username is added to existing user. Username can be added by user in setting page, added by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.username.removed

Occurs when username is removed from existing user. Username can be removed by user in setting page, removed by admin through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.username.updated

Occurs when username is updated. Username can be updated by user in setting page.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.oauth.connected

Occurs when user connected to a new OAuth provider.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.oauth.disconnected

Occurs when user disconnected from an OAuth provider. It can be done by user disconnected OAuth provider in the setting page, or admin removed OAuth identity through admin api or portal.

```javascript
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

## Check the webhook signatures

Each webhook event request is signed with a secret key shared between Authgear and the webhook handler. The developer must validate the signature and reject requests with invalid signature to ensure the request originates from Authgear.

The signature is calculated as the hex encoded value of HMAC-SHA256 of the request body and included in the header `x-authgear-body-signature`.

{% hint style="info" %}
Getting the secret key in the Portal is in our roadmap. [Track the issue here.](https://github.com/authgear/authgear-server/issues/912)
{% endhint %}

