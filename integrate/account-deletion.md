---
description: Allow end-users to initiate account deletion within the apps.
---

# Account Deletion

In Oct 2021, [Apple announced](https://developer.apple.com/news/?id=mdkbobfo) that all apps allowing users to create accounts should also provide ways for them to **initiate account deletion within the apps,** starting from January 31, 2022. It is also a good design to give your end-users more control over their data.

On Jan 22, 2022 [Apple decided](https://developer.apple.com/news/?id=i71db0mv) to extend the deadline to June 30 2022.

## Show "Delete Account" button in User Settings

In the pre-built [**User Settings**](auth-ui.md) page, you can show a button for the end-users to initiate account deletion.

Enable this button in the **Advanced** -> **Account Deletion** page in the **Portal**

!["Delete your account" button in the User Settings page](<../.gitbook/assets/Delete Your Account Button.jpg>)

Note that if you enable this feature, you have to prepare for encountering invalid session every time your users close User Settings.
If your users unfortunately decide to delete their account in User Settings, their sessions will become invalid.
You must verify the validity of the session every time User Settings is closed.
The following code snippet demonstrates this idea.

{% tabs %}
{% tab title="React Native" %}
```typescript
// This method blocks until the user closes User Settings.
await authgear.open(Page.Settings);
// One way to verify the validity of the session is to get User Info once.
await authgear.fetchUserInfo();
```
{% endtab %}

## Deactivated User

When the end-user has initiated the account deletion. Their account will be **deactivated** and scheduled for deletion after the grace period.

**Deactivated** users are always **disabled**. They will not be able to complete the authentication process. The `is_deactivated` status signal that the `is_disabled` status was turned `true` by the end-user themselves rather than the admin.

## Schedule Deletion

You can set the grace period for how long the user account will be deactivated before deleted from the system. The default value is 30 days, you can choose between 1 to 180 days.

## Initiate Deletion from the Portal

An end-user account can also be deleted using the **Portal**. In the **User Management** page, click the **Remove User** button to remove them immediately or schedule the deletion.

## Initiate Deletion from Admin API

Alternatively, if you do not enable the "Delete Account" button in User Settings, you can implement the button in your app by yourself.
Your backend server can invoke the mutation `scheduleAccountDeletion` to initiate the account deletion.

Here is an example of how to invoke the mutation.

```graphql
mutation {
  scheduleAccountDeletion(input: {
    userID: "USER_ID"
  }) {
    user {
      id
      isDisabled
      isDeactivated
      disableReason
      deleteAt
    }
  }
}
```

## Webhook events

You may listen to the following events to integrate the deletion behavior to your apps.

**Non-blocking events**

* `user.disabled`
* `user.reenabled`
* `user.deletion_scheduled`&#x20;
* `user.deletion_unscheduled`
* `user.deleted`

**Blocking event**

* `user.pre_schedule_deletion`

See the event details in [Webhooks](broken-reference).
