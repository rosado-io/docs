# Connect Apps to Microsoft AD FS

## Prerequisite

1. Setup your own AD FS server
2. Create an application in your AD FS Server, obtain "Client ID", "Client Secret" and "Discovery Document Endpoint". Discovery Document Endpoint typically ends with `/.well-known/openid-configuration`. Configure your application with redirect uri `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/adfs`.

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:alias`. The `alias` is used as the identifier of OAuth provider. You can configure the `alias` in Authgear Portal.
{% endhint %}

## Configure Sign in with Microsoft AD FS through the portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with Microsoft AD FS"
2. Fill in "Client ID", "Client Secret" and "Discovery Document Endpoint"
3. **Save** the settings.

🎉 Done! You have just added Microsoft AD FS Login to your apps!
