# User Profile

{% hint style="info" %}
Ability to get user details like their email addresses directly from the client API and the SDKs is on our roadmap. You can keep track of the issue here: [https://github.com/authgear/authgear-server/issues/260](https://github.com/authgear/authgear-server/issues/260)
{% endhint %}

## UserInfo Endpoint

The UserInfo endpoint returns the Claims about the authenticated end-user, including the standard profile and custom attributes.&#x20;

At the meantime, the `userInfo` object is returned from calling **fetch user info **function which contains a unique identifier of the user.

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

