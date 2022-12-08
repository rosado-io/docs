# Audit Log

Authgear provides the event logs for you to analyze security issues and monitor the business.

## View and retrieve logs

You can view the audit log in the Portal, or retrieve logs using the [Admin API](../apis/admin-api/).

### View in Portal

The portal provides an interface for you to look up the log by event and date range.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>View audit logs in the Portal</p></figcaption></figure>

### Retrieve with Admin API

The API schema can be found in the [Admin API QraphiQL Explorer](../apis/admin-api/#api-explorer). For example:

```graphql
query {
  auditLogs(first:5){
    edges{
      node{
        activityType
        clientID
        createdAt
        data
      }
    }
  }
}
```

## Log events

Here are the list of activity types that are logged:

#### Authentication failed

* AUTHENTICATION\_IDENTITY\_ANONYMOUS\_FAILED
* AUTHENTICATION\_IDENTITY\_BIOMETRIC\_FAILED
* AUTHENTICATION\_IDENTITY\_LOGIN\_ID\_FAILED
* AUTHENTICATION\_PRIMARY\_OOB\_OTP\_EMAIL\_FAILED
* AUTHENTICATION\_PRIMARY\_OOB\_OTP\_SMS\_FAILED
* AUTHENTICATION\_PRIMARY\_PASSWORD\_FAILED
* AUTHENTICATION\_SECONDARY\_OOB\_OTP\_EMAIL\_FAILED
* AUTHENTICATION\_SECONDARY\_OOB\_OTP\_SMS\_FAILED
* AUTHENTICATION\_SECONDARY\_PASSWORD\_FAILED
* AUTHENTICATION\_SECONDARY\_RECOVERY\_CODE\_FAILED
* AUTHENTICATION\_SECONDARY\_TOTP\_FAILED

#### Identity changes

* IDENTITY\_BIOMETRIC\_DISABLED
* IDENTITY\_BIOMETRIC\_ENABLED
* IDENTITY\_EMAIL\_ADDED
* IDENTITY\_EMAIL\_REMOVED
* IDENTITY\_EMAIL\_UPDATED
* IDENTITY\_OAUTH\_CONNECTED
* IDENTITY\_OAUTH\_DISCONNECTED
* IDENTITY\_PHONE\_ADDED
* IDENTITY\_PHONE\_REMOVED
* IDENTITY\_PHONE\_UPDATED
* IDENTITY\_USERNAME\_ADDED
* IDENTITY\_USERNAME\_REMOVED
* IDENTITY\_USERNAME\_UPDATED

#### User actions

* USER\_ANONYMOUS\_PROMOTED
* USER\_AUTHENTICATED
* USER\_CREATED
* USER\_DELETED
* USER\_DELETION\_SCHEDULED
* USER\_DELETION\_UNSCHEDULED
* USER\_DISABLED
* USER\_PROFILE\_UPDATED
* USER\_REENABLED
* USER\_SESSION\_TERMINATED
* USER\_SIGNED\_OUT

#### Others

* WHATSAPP\_OTP\_VERIFIED
* SMS\_SENT
* EMAIL\_SENT

## Log data

Each audit log event contains the following attributes in their data

| Attribute | Description                                                                                                                                                                                                                                                            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`      | Unique identifier of the event                                                                                                                                                                                                                                         |
| `seq`     | Sequence number of the event                                                                                                                                                                                                                                           |
| `type`    | Activity type                                                                                                                                                                                                                                                          |
| `context` | The who, when and where of the event triggered. e.g. IP address, user agent, user ID, timestamp                                                                                                                                                                        |
| `payload` | <p>Relevant data according to the event type:</p><p><strong>Messaging (SMS, Email OTP):</strong> the phone number/email address of the receiver</p><p><strong>Authentication/Identity/User actions:</strong> a snapshot of the related session and user attributes</p> |
