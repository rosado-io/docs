# Social login with Linkedin

## Prerequisite

1. Create an app [here](https://www.linkedin.com/developers/)
2. In the "Products" section, choose "Sign In with LinkedIn"
3. In the details page of the created app, click "Auth" tab
4. Take notes of "Client ID" and "Client Secret", add `{Authgear domain}/sso/oauth2/callback/{alias}` to "Redirect URLs" in "OAuth 2.0 settings" section \(`alias` can be configured in Authgear portal\)

## Configure Sign in with LinkedIn through the portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with LinkedIn"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID" and "Client Secret"
4. Scroll down and click save

## Notes

* "alias" is used as the identifier of OAuth provider
* Redirect URI has the form "/sso/oauth2/callback/:alias"
