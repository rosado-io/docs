# Facebook

## Prerequisite

To configure "Login with Facebook" for Authgear, you will need the following

1. A [Facebook Developer Account](https://developers.facebook.com/apps/)
2. A [registered Facebook App](https://developers.facebook.com/docs/apps#register)
3. Go to app dashboard, click plus button next to "Product" in the sidebar, add "Facebook Login".
4. Then go to "Settings" of "Facebook Login", add `https://{your domain}/sso/oauth2/callback/{alias}` to "Valid OAuth Redirect URIs" \(`alias` can be configured in Authgear portal\)

## Configure Login with Facebook through the portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Login with Facebook"
2. In the app dashboard of your app in the Facebook developer portal, click "Settings" in the sidebar and click "Basic"
3. In Authgear portal, fill in "Alias" with alias used in redirect URI
4. Fill in "Client ID" with "App ID"
5. Fill in "Client Secret" with "App Secret"
6. Scroll to the bottom and click save

## Notes

* "alias" is used as the identifier of OAuth provider
* Redirect URI has the form "/sso/oauth2/callback/:alias"

