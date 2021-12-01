# User Profile

{% hint style="info" %}
The user profile features, which include Standard Attributes and Custom Attributes is under development. You can keep track of the issue here: [https://github.com/authgear/authgear-server/issues/1510](https://github.com/authgear/authgear-server/issues/1510)
{% endhint %}

The user profiles contain information about your end-users such as name, email, addresses, and unique identifier. You can manage the profiles via the Portal & Admin API. The end-users can also manage their own profile through the Profile section in the [User Setting page](auth-ui.md) provided by the AuthUI.

## UserInfo Endpoint

The UserInfo endpoint returns the Claims about the authenticated end-user, including the standard profile and custom attributes.&#x20;

In the meantime, the `userInfo` object is returned from calling **fetch user info** function which contains a unique identifier of the user.

| Key         | Type      | Description                                                                                                                                                                                                     |
| ----------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| isAnonymous | _boolean_ | Indicate if the user is anonymous, i.e. no [identity](../strategies/user-identity-and-authenticator.md#identity) or [authenticator](../strategies/user-identity-and-authenticator.md#authenticator) is provided |
| isVerified  | _boolean_ | Indicate if the user completed the verification requirement                                                                                                                                                     |
| sub         | _string_  | Unique identifier of the user in your Authgear project                                                                                                                                                          |

{% tabs %}
{% tab title="JavaScript" %}
```javascript
try{
    const userInfo = await authgear.fetchUserInfo()
} catch(e) {
    // failed to fetch user info
    // the refresh token maybe expired or revoked
}
```
{% endtab %}

{% tab title="iOS" %}
```swift
authgear.fetchUserInfo { userInfoResult in
    // sessionState is now up to date
    // it will change to .noSession if the session is invalid
    let sessionState = authgear.sessionState

    switch userInfoResult {
    case let .success(userInfo):
        // read the userInfo if needed
    case let .failure(error):
        // failed to fetch user info
        // the refresh token maybe expired or revoked
}
```
{% endtab %}

{% tab title="Android" %}
```kotlin
authgear.fetchUserInfo(new OnFetchUserInfoListener() {
    @Override
    public void onFetchedUserInfo(@NonNull UserInfo userInfo) {
        // sessionState is now up to date
        // read the userInfo if needed
    }

    @Override
    public void onFetchingUserInfoFailed(@NonNull Throwable throwable) {
        // sessionState is now up to date
        // it will change to NO_SESSION if the session is invalid
    }
});
```
{% endtab %}
{% endtabs %}

## Standard Attributes

The following attributes are built-in supported by Authgear. They are the set of [**Standard Claims** defined by the OIDC specifications](https://openid.net/specs/openid-connect-core-1\_0.html#StandardClaims). Some of them are default hidden from the Admin Portal and end-users. Their visibility and mutability can be configured through the Admin Portal.

| Attribute name | Default Visibility | Format                                                                                     |
| -------------- | ------------------ | ------------------------------------------------------------------------------------------ |
| Name           | Hidden             | String                                                                                     |
| Given Name     | Editable           | String                                                                                     |
| Family Name    | Editable           | String                                                                                     |
| Middle Name    | Hidden             | String                                                                                     |
| Nickname       | Hidden             | String                                                                                     |
| Profile        | Hidden             | URL String                                                                                 |
| Picture        | Editable           | URL String                                                                                 |
| Website        | Hidden             | URL String                                                                                 |
| Gender         | Editable           | `male`, `female` or Custom String                                                          |
| Birthdate      | Editable           | Date in YYYY-MM-DD                                                                         |
| Timezone       | Editable           | [tz database zone name](https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones) |
| Language       | Editable           | BCP47 language tag enabled by the project                                                  |
| Address        | Hidden             | JSON Object                                                                                |

### Standard Attributes that are coupled with Identities

The following attributes are coupled with the [identities](../strategies/user-identity-and-authenticator.md#identity) owned by the end-user. The represents the email addresses, phone numbers, or usernames the end-users are using to authenticate themselves on Authgear. If the end-user uses a third-party identity provider for authentication, these attributes will be coupled with the corresponding attributes returned by the provider.

* `email`
* `email_verified`
* `phone_number`
* `phone_number_verified`
* `preferred_username`

## User Profile Configuration

The access rights for different parties on individual attributes can be configured through the Authgear Portal. Under the hood, all the attributes are available, however, they can be configured to be `hidden` or `read-only` according to the needs of your projects to avoid confusion.

These are the parties that have access to the user profile:

### The Admin API

Through [the Admin API](../apis/admin-apis.md), developers **ALWAYS** have **full access** to **ALL** the standard attributes and custom attributes. The Admin API allows the developer to view or edit the standard attributes and the custom attributes.&#x20;

### The Portal

The admin user can view or edit the standard attributes via the Authgear Portal.

### The Session Bearer

The session bearer is someone who has a valid session cookie or a valid access token. The standard attributes of the end-user whom the session represents can be viewed by accessing [the UserInfo endpoint](user-profile.md#userinfo-endpoint) and [the resolver endpoint](https://docs.authgear.com/get-started/backend-integration/nginx). The session bearer can be the end-user, the client mobile app, or the client website.&#x20;

### The End-user

The end-user can view or edit the standard attributes through the Profile section in the [User Setting page](auth-ui.md) provided by the AuthUI.

## Profiles from Third-party Identity Providers

Authgear supports various [social and enterprise identity providers](../strategies/how-to-setup-sso-integrations/). End-users can sign up and log in to your apps via these connections. Upon signup, these providers will return a set of user attributes about the end-user. Authgear will copy those attributes and populate the profile of the end-user.

More info about the population logic can be found in [the specification](https://github.com/authgear/authgear-server/blob/master/docs/specs/user-profile/design.md#standard-attributes-population).
