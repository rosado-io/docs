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
1. In the portal, go to **Advanced** > **Webhooks**.
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

* [user.pre\_create](webhooks.md#user.pre\_create)
* [user.profile.pre\_update](webhooks.md#user.profile.pre\_update)
* [user.pre\_schedule\_deletion](webhooks.md#user.pre\_schedule\_deletion)

#### Non-blocking Events

* [user.created](webhooks.md#user.created)
* [user.profile.updated](webhooks.md#user.profile.updated)
* [user.authenticated](webhooks.md#user.authenticated)
* [user.disabled](webhooks.md#user.disabled)
* [user.reenabled](webhooks.md#user.reenabled)
* [user.anonymous.promoted](webhooks.md#user.anonymous.promoted)
* [user.deletion\_scheduled](webhooks.md#user.deletion\_scheduled)
* [user.deletion\_unscheduled](webhooks.md#user.deletion\_unscheduled)
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
* [identity.biometric.enabled](webhooks.md#identity.biometric.enabled)
* [identity.biometric.disabled](webhooks.md#identity.biometric.disabled)

#### user.pre\_create

Occurs right before the user creation. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or API. The operation can be aborted by providing a specific response in your webhook, details see [Webhook Blocking Events](webhooks.md#blocking-events).

```json
{
  "type": "user.pre_create",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identities": [
      {
        "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
        "created_at": "2006-01-02T03:04:05.123456Z",
        "updated_at": "2006-01-02T03:04:05.123456Z",
        "type": "login_id",
        "claims": {
          "email": "user@example.com",
          "https://authgear.com/claims/login_id/key": "email",
          "https://authgear.com/claims/login_id/original_value": "user@example.com",
          "https://authgear.com/claims/login_id/type": "email",
          "https://authgear.com/claims/login_id/value": "user@example.com"
        }
      }
    ]
  }
}
```

#### user.profile.pre\_update

Occurs right before the update of the user profile.

```json
{
  "type": "user.profile.pre_update",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "name": "Chris",
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.pre\_schedule\_deletion

Occurs right before account deletion is scheduled.

```json
{
  "type": "user.pre_schedule_deletion",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": true,
      "delete_at": "2022-09-30T15:18:19.040081Z",
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.created

Occurs after a new user is created. User can be created by user signup, user signup as an anonymous user, or created by the admin via the Portal or API.

```json
{
  "type": "user.created",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identities": [
      {
        "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
        "created_at": "2006-01-02T03:04:05.123456Z",
        "updated_at": "2006-01-02T03:04:05.123456Z",
        "type": "login_id",
        "claims": {
          "email": "user@example.com",
          "https://authgear.com/claims/login_id/key": "email",
          "https://authgear.com/claims/login_id/original_value": "user@example.com",
          "https://authgear.com/claims/login_id/type": "email",
          "https://authgear.com/claims/login_id/value": "user@example.com"
        }
      }
    ]
  }
}
```

#### user.profile.updated

Occurs when the user profile is updated.

```json
{
  "type": "user.profile.updated",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "name": "Chris",
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.authenticated

Occurs after the user logged in.

```json
{
  "type": "user.authenticated",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "session": {
      "id": "6e6e5d9b-7f85-4a8f-a157-2de94694dfea",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "idp",
      "amr": ["pwd"],
      "lastAccessedAt": "2006-01-02T03:04:05.123456Z",
      "createdByIP": "127.0.0.1",
      "lastAccessedByIP": "127.0.0.1",
      "lastAccessedByIPCountryCode": "",
      "lastAccessedByIPEnglishCountryName": "",
      "displayName": "Chrome 104.0.0"
    }
  }
}
```

#### user.disabled

Occurs when the user was disabled.

```json5
{
  "type": "user.disabled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.reenabled

Occurs when the user was re-enabled.

```json5
{
  "type": "user.reenabled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.anonymous.promoted

Occurs whenever an anonymous user is promoted to a normal user.

```json
{
  "type": "user.anonymous.promoted",
  "payload": {
    "anonymous_user": {
      "id": "7a009f88-c636-4245-91ec-7b174dc6a1a1",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "user": {
      "id": "7a009f88-c636-4245-91ec-7b174dc6a1a1",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identities": [
      {
        "id": "1450234b-5e2c-4aa0-b400-f2d9d70a2f45",
        "created_at": "2006-01-02T03:04:05.123456Z",
        "updated_at": "2006-01-02T03:04:05.123456Z",
        "type": "login_id",
        "claims": {
          "email": "user@example.com",
          "https://authgear.com/claims/login_id/key": "email",
          "https://authgear.com/claims/login_id/original_value": "user@example.com",
          "https://authgear.com/claims/login_id/type": "email",
          "https://authgear.com/claims/login_id/value": "user@example.com"
        }
      }
    ]
  }
}
```

#### user.deletion\_scheduled

Occurs when an account deletion was scheduled.

```json5
{
  "type": "user.deletion_scheduled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": true,
      "is_deactivated": true,
      "delete_at": "2006-01-02T03:04:05.123456Z",
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.deletion\_unscheduled

Occurs when an account deletion was unscheduled.

```json5
{
  "type": "user.deletion_unscheduled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### user.deleted

Occurs when the user was deleted.

```json5
{
  "type": "user.deleted",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": false,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": false,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": false,
        "updated_at": 1136171045
      }
    }
  }
}
```

#### identity.email.added

Occurs when a new email is added to an existing user. Email can be added by the user in the setting page, added by the admin through the admin API or Portal.

```json
{
  "type": "identity.email.added",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "a0c55481-147e-4a58-876e-10dffedfd5cd",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/original_value": "user@example.com",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user@example.com"
      }
    }
  }
}
```

#### identity.email.removed

Occurs when an email address is removed from an existing user. Email can be removed by the user in the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.email.removed",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/original_value": "user@example.com",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user@example.com"
      }
    }
  }
}
```

#### identity.email.updated

Occurs when an email address is updated. Email can be updated by the user on the setting page.

```json
{
  "type": "user.email.updated",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user3@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user3@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/original_value": "user3@example.com",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user3@example.com"
      }
    },
    "old_identity": {
      "id": "239d585d-9b90-4148-9aa2-2e3131b5847a",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "email": "user@example.com",
        "https://authgear.com/claims/login_id/key": "email",
        "https://authgear.com/claims/login_id/original_value": "user@example.com",
        "https://authgear.com/claims/login_id/type": "email",
        "https://authgear.com/claims/login_id/value": "user@example.com"
      }
    }
  }
}
```

#### identity.phone.added

Occurs when a new phone number is added to an existing user. Phone numbers can be added by the user in the setting page, added by admin through admin API or Portal.

```json
{
  "type": "user.phone.added",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "phone_number": "+447400123456",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123456",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123456",
        "phone_number": "+447400123456"
      }
    }
  }
}
```

#### identity.phone.removed

Occurs when a phone number is removed from an existing user. Phone numbers can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.phone.removed",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123455",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    }
  }
}
```

#### identity.phone.updated

Occurs when a phone number is updated. Phone numbers can be updated by the user on the setting page.

```json
{
  "type": "identity.phone.updated",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "phone_number": "+447400123455",
        "phone_number_verified": true,
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123455",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123455",
        "phone_number": "+447400123455"
      }
    },
    "old_identity": {
      "id": "9fa5668d-a796-4817-93e1-d4096e5966ac",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "phone",
        "https://authgear.com/claims/login_id/original_value": "+447400123456",
        "https://authgear.com/claims/login_id/type": "phone",
        "https://authgear.com/claims/login_id/value": "+447400123456",
        "phone_number": "+447400123456"
      }
    }
  }
}

```

#### identity.username.added

Occurs when a new username is added to an existing user. Username can be added by the user in setting page, added by admin through Admin API or Portal.

```json
{
  "type": "identity.username.added",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "preferred_username": "user01",
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user01",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user01",
        "preferred_username": "user01"
      }
    }
  }
}

```

#### identity.username.removed

Occurs when the username is removed from an existing user. The username can be removed by the user on the setting page, removed by admin through admin API or Portal.

```json
{
  "type": "identity.username.removed",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user02",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user02",
        "preferred_username": "user02"
      }
    }
  }
}
```

#### identity.username.updated

Occurs when the username is updated. The username can be updated by the user on the setting page.

```json
{
  "type": "identity.username.updated",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "preferred_username": "user02",
        "updated_at": 1136171045
      }
    },
    "new_identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user02",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user02",
        "preferred_username": "user02"
      }
    },
    "old_identity": {
      "id": "38683500-f6ce-477d-b944-1f915a451995",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "login_id",
      "claims": {
        "https://authgear.com/claims/login_id/key": "username",
        "https://authgear.com/claims/login_id/original_value": "user01",
        "https://authgear.com/claims/login_id/type": "username",
        "https://authgear.com/claims/login_id/value": "user01",
        "preferred_username": "user01"
      }
    }
  }
}
```

#### identity.oauth.connected

Occurs when a user has connected to a new OAuth provider.

```json
{
  "type": "identity.oauth.connected",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "f22425cf-68b2-45f8-936d-3c54c9cdf5c7",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "oauth",
      "claims": {
        "email": "xxx@gmail.com",
        "email_verified": true,
        "family_name": "",
        "given_name": "",
        "https://authgear.com/claims/oauth/profile": {
          "at_hash": "",
          "aud": ["xxx.apps.googleusercontent.com"],
          "azp": "xxx.apps.googleusercontent.com",
          "email": "xxx@gmail.com",
          "email_verified": true,
          "exp": "2006-01-02T03:04:05Z",
          "family_name": "",
          "given_name": "",
          "iat": "2006-01-02T03:04:05Z",
          "iss": "https://accounts.google.com",
          "locale": "en",
          "name": "",
          "nonce": "",
          "picture": "https://lh3.googleusercontent.com/a/xxx",
          "sub": ""
        },
        "https://authgear.com/claims/oauth/provider_alias": "google",
        "https://authgear.com/claims/oauth/provider_type": "google",
        "https://authgear.com/claims/oauth/subject_id": "",
        "locale": "en",
        "name": "",
        "picture": "https://lh3.googleusercontent.com/a/xxx"
      }
    }
  }
}

```

#### identity.oauth.disconnected

Occurs when a user is disconnected from an OAuth provider. It can be done by the user disconnecting their OAuth provider in the setting page, or the admin removing the OAuth identity through admin API or Portal.

```json
{
  "type": "identity.oauth.disconnected",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "f22425cf-68b2-45f8-936d-3c54c9cdf5c7",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "oauth",
      "claims": {
        "email": "xxx@gmail.com",
        "email_verified": true,
        "family_name": "",
        "given_name": "",
        "https://authgear.com/claims/oauth/profile": {
          "at_hash": "",
          "aud": ["xxx.apps.googleusercontent.com"],
          "azp": "xxx.apps.googleusercontent.com",
          "email": "xxx@gmail.com",
          "email_verified": true,
          "exp": "2006-01-02T03:04:05Z",
          "family_name": "",
          "given_name": "",
          "iat": "2006-01-02T03:04:05Z",
          "iss": "https://accounts.google.com",
          "locale": "en",
          "name": "",
          "nonce": "",
          "picture": "https://lh3.googleusercontent.com/a/xxx",
          "sub": ""
        },
        "https://authgear.com/claims/oauth/provider_alias": "google",
        "https://authgear.com/claims/oauth/provider_type": "google",
        "https://authgear.com/claims/oauth/subject_id": "",
        "locale": "en",
        "name": "",
        "picture": "https://lh3.googleusercontent.com/a/xxx"
      }
    }
  }
}
```

#### identity.biometric.enabled

Occurs when the user enabled biometric login.

```json
{
  "type": "identity.biometric.enabled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "5cb77960-634b-4c0e-8a0e-6c2c73fb8f47",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "biometric",
      "claims": {
        "https://authgear.com/claims/biometric/device_info": {
          "ios": {
            "NSBundle": {
              "CFBundleDisplayName": "Authgear demo iOS",
              "CFBundleExecutable": "ios_example",
              "CFBundleIdentifier": "com.authgear.exampleapp.ios",
              "CFBundleName": "ios_example",
              "CFBundleShortVersionString": "1.0",
              "CFBundleVersion": "1653565975"
            },
            "NSProcessInfo": {
              "isMacCatalystApp": false,
              "isiOSAppOnMac": false
            },
            "UIDevice": {
              "model": "iPhone",
              "name": "iPhone",
              "systemName": "iOS",
              "systemVersion": "16.1.1",
              "userInterfaceIdiom": "phone"
            },
            "uname": {
              "machine": "iPhone13,3",
              "nodename": "Users-iPhone",
              "release": "22.1.0",
              "sysname": "Darwin",
              "version": "Darwin Kernel Version 22.1.0: Thu Oct  6 19:34:22 PDT 2022; root:xnu-8792.42.7~1/RELEASE_ARM64_T8101"
            }
          }
        },
        "https://authgear.com/claims/biometric/formatted_device_info": "iPhone 12 Pro",
        "https://authgear.com/claims/biometric/key_id": "1CC9D95A-6578-4557-8279-C4D5699D3549"
      }
    }
  }
}
```

#### identity.biometric.disabled

Occurs when biometric login is disabled. It will be triggered only when the user disabled it from the settings page or the admin disabled it from the admin api or portal.

```json
{
  "type": "identity.biometric.disabled",
  "payload": {
    "user": {
      "id": "338deafa-400b-4589-a922-2c92d670b757",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "last_login_at": "2006-01-02T03:04:05.123456Z",
      "is_anonymous": false,
      "is_verified": true,
      "is_disabled": false,
      "is_deactivated": false,
      "can_reauthenticate": true,
      "standard_attributes": {
        "email": "user@example.com",
        "email_verified": true,
        "updated_at": 1136171045
      }
    },
    "identity": {
      "id": "5cb77960-634b-4c0e-8a0e-6c2c73fb8f47",
      "created_at": "2006-01-02T03:04:05.123456Z",
      "updated_at": "2006-01-02T03:04:05.123456Z",
      "type": "biometric",
      "claims": {
        "https://authgear.com/claims/biometric/device_info": {
          "ios": {
            "NSBundle": {
              "CFBundleDisplayName": "Authgear demo iOS",
              "CFBundleExecutable": "ios_example",
              "CFBundleIdentifier": "com.authgear.exampleapp.ios",
              "CFBundleName": "ios_example",
              "CFBundleShortVersionString": "1.0",
              "CFBundleVersion": "1653565975"
            },
            "NSProcessInfo": {
              "isMacCatalystApp": false,
              "isiOSAppOnMac": false
            },
            "UIDevice": {
              "model": "iPhone",
              "name": "iPhone",
              "systemName": "iOS",
              "systemVersion": "16.1.1",
              "userInterfaceIdiom": "phone"
            },
            "uname": {
              "machine": "iPhone13,3",
              "nodename": "Users-iPhone",
              "release": "22.1.0",
              "sysname": "Darwin",
              "version": "Darwin Kernel Version 22.1.0: Thu Oct  6 19:34:22 PDT 2022; root:xnu-8792.42.7~1/RELEASE_ARM64_T8101"
            }
          }
        },
        "https://authgear.com/claims/biometric/formatted_device_info": "iPhone 12 Pro",
        "https://authgear.com/claims/biometric/key_id": "1CC9D95A-6578-4557-8279-C4D5699D3549"
      }
    }
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

## Payload Reference

The payloads in the webhook events may contain different objects. You can refer to the below for their attributes.

### User Object

`user` in payload have the following attributes.

```json5
"payload":{
  "user": {
    "id": "string",
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "last_login_at": "timestamp",
    "is_anonymous": boolean,
    "is_verified": boolean,
    "is_disabled": boolean,
    "is_deactivated": boolean,
    "can_reauthenticate": boolean,
    "standard_attributes": {
      ...
    },
    "custom_attributes": {
      ...
    }
  }
}
```
