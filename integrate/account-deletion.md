---
description: Allow end-users to initiate account deletion within the apps.
---

# Account Deletion

In Oct 2021, [Apple announced](https://developer.apple.com/news/?id=mdkbobfo) that all apps allowing users to create accounts should also provide ways for them to **initiate account deletion within the apps,** starting from January 31, 2022. It is also a good design to give your end-users more control over their data.

{% hint style="info" %}
This feature is WIP. You can track the progress in [this GitHub issue](https://github.com/authgear/authgear-server/issues/1736).
{% endhint %}

## Show "Delete Account" button in User Settings

In the pre-built [**User Settings**](auth-ui.md) page, you can show a button for the end-users to initiate account deletion.

Enable this button in the **Account Deletion** page in the **Portal**

!["Delete your account" button in the User Settings page](<../.gitbook/assets/Delete Your Account Button.jpg>)

## Deactivated User

When the end-user has initiated the account deletion. Their account will be **deactivated** and scheduled for deletion after the grace period.

**Deactivated** users are always **disabled**. They will not be able to complete the authentication process. The `is_deactivated` status signal that the `is_disabled` status was turned `true` by the end-user themselves rather than the admin.

## Schedule Deletion

You can set the grace period for how long the user account will be deactivated before deleted from the system. The default value is 30 days, you can choose between 1 to 180 days.

## Initiate Deletion from the Portal

An end-user account can also be deleted using the **Portal**. In the **User Management** page, click the **Remove User** button to remove them immediately or schedule the deletion.

## Initiate Deletion from Admin API

```graphql
{
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
