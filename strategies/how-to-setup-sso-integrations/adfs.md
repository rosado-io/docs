# Connect Apps to Microsoft AD FS

## Prerequisite

1. Setup your own AD FS server
2. Create an application in your AD FS Server, obtain "Client ID", "Client Secret" and "Discovery Document Endpoint". Discovery Document Endpoint typically ends with `/.well-known/openid-configuration`. Configure your application with redirect uri `https://{your domain}/sso/oauth2/callback/{alias}`.

## Configure Sign in with Microsoft AD FS through the portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with Microsoft AD FS"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID", "Client Secret" and "Discovery Document Endpoint"
4. Click save

## Notes

* "alias" is used as the identifier of OAuth provider
* Redirect URI has the form "/sso/oauth2/callback/:alias"

