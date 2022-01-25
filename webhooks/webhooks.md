---
description: >-
  Authgear can deliver webhook events to your server when important events such
  as new user signup happen.
---

# Webhooks

Authgear uses webhooks to notify your application server when events happen in your Authgear project. To use webhooks you will need to:

1. Create a webhook handler endpoint in your application server.
2. Configure Authgear to send events to your webhook endpoint.

There are two kinds of events, **Blocking** and **Non-blocking** events.

* Blocking event is triggered before the operation is performed. The operation can be aborted by your webhook endpoint response.
* Non-blocking event is triggered after the operation is performed.

## Configure Authgear to send events to your webhook endpoint

{% tabs %}
{% tab title="Portal" %}
1. In the portal, go to **Settings** > **Webhooks**.
2. Add your webhook endpoints in **Blocking Events** and **Non-Blocking Events**, depending on which event you want to listen to.
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

Webhook handlers must be in **HTTPS**. This ensures the integrity and confidentiality of the delivery.

The webhook handler endpoint must be an absolute URL. The webhook events are sent to the webhook handler endpoint via `POST` requests. The webhook handlers must return a status code within the `2xx` range. Other status codes are considered as a failed delivery.

Each event can have multiple handlers. The order of delivery is unspecified for non-blocking events. Blocking events are delivered in the source order as in the configuration.

## Blocking Events

Blocking events are delivered to webhook handlers synchronously, right before committing changes to the database.

Webhook handler must respond with a JSON body to indicate whether the operation should continue.

To let the operation proceed, respond with `is_allowed` being set to `true`.

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

Each blocking event webhook handler must respond within 5 seconds. And all blocking handlers in total should take less than 10 seconds to complete. Otherwise, it would be considered a failed delivery.

### Webhook Blocking Event Mutations

The webhook handler can optionally mutate the object in the payload. The webhook handlers should specify the mutations in their responses. If the operation is failed, i.e. `"is_allowed": false` , the mutation will be discarded.

Given the event

```json
{
  "payload": {
    "user": {
      "standard_attributes": {
        "name": "John"
      }
    }
  }
}
```

The webhook handler can mutate the user with the following response.

```json
{
  "is_allowed": true,
  "mutations": {
    "user": {
      "standard_attributes": {
        "name": "Jane"
      }
    }
  }
}
```

Objects not appearing in `mutations` are left intact. The mutated objects will NOT be merged with the original ones.

The mutated payloads are NOT validated and are propagated along the handler chain. The payload will only be validated after traversing the handler chain.

Mutations do NOT generate extra events to avoid infinite loops.

Currently, only the `standard_attributes` of the user object are mutable.

## Non-blocking Events

Non-blocking events are delivered to webhook handlers asynchronously after the operation is performed (i.e. committed into the database).

The webhook handlers must respond with a status code within the `2xx` range within 60 seconds. Otherwise, it would be considered a failed delivery.

The response body of a non-blocking event webhook handler is ignored.

## Webhook Request Body

All webhook events have the following shape:

```json
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
* `type`: The type of webhook event.
* `payload`: The payload of the webhook event, varies with type.
* `context`: The context of the webhook event.

### Webhook Event Context

* `timestamp`: signed 64-bit UNIX timestamp of when this event is generated. Retried deliveries do not affect this field.
* `user_id`: The ID of the user associated with the event. It may be absent. For example, the user has not been authenticated yet.
* `preferred_languages`: User preferred languages, which are inferred from the request. Return values of the `ui_locales` query if it is provided in the Auth UI, otherwise return languages in the `Accept-Language` request header.
* `language`: User locale which is derived based on user's preferred languages and app's languages config.
* `triggered_by`: Triggered by indicates who triggered the events, values can be `user` or `admin_api`. `user` means it is triggered by user in auth ui. `admin_api` means it is triggered by admin api or admin portal.

## Webhook Event List

#### Blocking Events

* [user.pre\_create](webhooks.md#user.pre_create)
* [user.profile.pre\_update](webhooks.md#user.profile.pre_update)
* [user.pre\_schedule\_deletion](webhooks.md#user.pre_schedule_deletion)

#### Non-blocking Events

* [user.created](webhooks.md#user.created)
* [user.profile.updated](webhooks.md#user.profile.updated)
* [user.authenticated](webhooks.md#user.authenticated)
* [user.disabled](webhooks.md#user.disabled)
* [user.reenabled](webhooks.md#user.reenabled)
* [user.anonymous.promoted](webhooks.md#user.anonymous.promoted)
* [user.deletion\_scheduled](webhooks.md#user.deletion_scheduled)
* [user.deletion\_unscheduled](webhooks.md#user.deletion_unscheduled)
* [user.deleted](webhooks.md#user.deleted)
* [identity.email.added](webhooks.md#identity.email.added)
* [identity.email.removed](webhooks.md#identity.email.removed)
* [identity.email.updated](webhooks.md#identity.email.updated)
* [identity.phone.added](webhooks.md#identity.phone.added)
* [identity.phone.removed](webhooks.md#identity.phone.removed)
* [identity.phone.updated](webhooks.md#identity.phone.updated)
* [identity.username.added](webhooks.md#identity.username.added)
* [identity.username.removed](webhooks.md#identity.username.removed)
* [identity.username.updated](webhooks.md#identity.username.updated)
* [identity.oauth.connected](webhooks.md#identity.oauth.connected)
* [identity.oauth.disconnected](webhooks.md#identity.oauth.disconnected)

#### user.pre\_create

Occurs right before the user creation. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or API. The operation can be aborted by providing a specific response in your webhook, details see [Webhook Blocking Events](webhooks.md#blocking-events).

```json
{
  "payload": {
    "user": { /* ... */ },
    "identities": [ { /* ... */ } ]
  }
}
```

#### user.profile.pre\_update

Occurs right before the update of the user profile.

```json
{
  "payload": {
    "user": {
      "standard_attributes": {
        /* ... */ 
      }
      /* ... */ 
    }
  }
}
```

#### user.pre\_schedule\_deletion

Occurs right before account deletion is scheduled.

```json
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### user.created

Occurs after a new user is created. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or API.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identities": [ { /* ... */ } ]
  }
}
```

#### user.profile.updated

Occurs when the user profile is updated.

```json
{
  "payload": {
    "user" : { 
      "standard_attributes": {
        /* ... */ 
      }
      /* ... */ 
    }
  }
}
```

#### user.authenticated

Occurs after the user logged in.

```json
{
  "payload": {
    "user": { /* ... */ },
    "session": { /* ... */ }
  }
}
```

#### user.disabled

Occurs when the user was disabled.

```json5
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### user.reenabled

Occurs when the user was re-enabled.

```json5
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### user.anonymous.promoted

Occurs whenever an anonymous user is promoted to a normal user.

```json
{
  "payload": {
    "anonymous_user": { /* ... */ },
    "user": { /* ... */ },
    "identities": [{ /* ... */ }]
  }
}
```

#### user.deletion\_scheduled

Occurs when an account deletion was scheduled.

```json5
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### user.deletion\_unscheduled

Occurs when an account deletion was unscheduled.

```json5
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### user.deleted

Occurs when the user was deleted.

```json5
{
  "payload": {
    "user": { /* ... */ }
  }
}
```

#### identity.email.added

Occurs when a new email is added to an existing user. Email can be added by the user in the setting page, added by the admin through the admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.email.removed

Occurs when an email address is removed from an existing user. Email can be removed by the user in the setting page, removed by admin through admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.email.updated

Occurs when an email address is updated. Email can be updated by the user on the setting page.

```json
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.phone.added

Occurs when a new phone number is added to an existing user. Phone numbers can be added by the user in the setting page, added by admin through admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.phone.removed

Occurs when a phone number is removed from an existing user. Phone numbers can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.phone.updated

Occurs when a phone number is updated. Phone numbers can be updated by the user on the setting page.

```json
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.username.added

Occurs when a new username is added to an existing user. Username can be added by the user in setting page, added by admin through Admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.username.removed

Occurs when the username is removed from an existing user. The username can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.username.updated

Occurs when the username is updated. The username can be updated by the user on the setting page.

```json
{
  "payload": {
    "user": { /* ... */ },
    "old_identity": { /* ... */ },
    "new_identity": { /* ... */ }
  }
}
```

#### identity.oauth.connected

Occurs when a user has connected to a new OAuth provider.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

#### identity.oauth.disconnected

Occurs when a user is disconnected from an OAuth provider. It can be done by the user disconnecting their OAuth provider in the setting page, or the admin removing the OAuth identity through admin API or Portal.

```json
{
  "payload": {
    "user": { /* ... */ },
    "identity": { /* ... */ }
  }
}
```

## Check the webhook signatures

Each webhook event request is signed with a secret key shared between Authgear and the webhook handler. The developer must validate the signature and reject requests with invalid signatures to ensure the request originates from Authgear.

The signature is calculated as the hex encoded value of HMAC-SHA256 of the request body and included in the header `x-authgear-body-signature`.

To obtain the secret key, visit the portal and go to **Advanced** -> **Webhooks** -> **Webhook Signature**. You may need to reauthenticate yourselves before you can reveal the secret key.

Here is the sample code of how to calculate the signature and verify it.

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "crypto/subtle"
    "encoding/hex"
    "fmt"
    "io"
    "net/http"
)

// Obtain the secret in the portal.
const Secret = "SECRET"

// HMACSHA256String returns the hex-encoded string of HMAC-SHA256 code of body using secret as key.
func HMACSHA256String(data []byte, secret []byte) (sig string) {
    hasher := hmac.New(sha256.New, secret)
    _, _ = hasher.Write(data)
    signature := hasher.Sum(nil)
    sig = hex.EncodeToString(signature)
    return
}

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        b, err := io.ReadAll(r.Body)
        if err != nil {
            // Handle the error properly
            panic(err)
        }
        defer r.Body.Close()

        sigInHeader := []byte(r.Header.Get("X-Authgear-Body-Signature"))
        sig := []byte(HMACSHA256String(b, []byte(Secret)))

        // Prefer constant time comparison over == operator.
        if subtle.ConstantTimeCompare(sigInHeader, sig) != 1 {
            // The signature does not match
            // Do NOT trust the content of this webhook!!!
            panic(fmt.Errorf("%v != %v", string(sigInHeader), string(sig)))
        }

        // Continue your logic here.
    })
    http.ListenAndServe(":9999", nil)
}
```
{% endtab %}
{% endtabs %}
