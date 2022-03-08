# Connect Apps to Azure Active Directory

## Prerequisite

1. Create an Azure Active Directory (Azure AD) account [here](https://azure.microsoft.com/free)
2. Setup a tenant by completing [Quickstart: Set up a tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-create-new-tenant)
3. Register an application by completing [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
4.  Choose "Supported account type", the following options are supported:

    * Accounts in this organizational directory only (Contoso AD (dev) only - Single tenant)
    * Accounts in this organizational directory (Any Azure AD directory - Multitenant)
    * Accounts in this organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)

    "Personal Microsoft accounts only" is not supported yet. Remember the account type chosen as this affects the configuration on Authgear portal
5. Configure "Redirect URI" with `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/azureadv2`
6. Follow [this](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-a-client-secret) section to add a client secret. Remember to record the secret value when you add the client secret, as it will not be displayed again. This will be needed for configure OAuth client in Authgear.

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:alias`. The `alias` is used as the identifier of OAuth provider. You can configure the `alias` in Authgear Portal.
{% endhint %}

## Configure Sign in with Microsoft through the portal

In the Azure portal, go to the details page of your app, you can find:

1. Application (client) ID
2. Directory (tenant) ID

Then in Authgear portal, go to "Single-Sign On" page, and do the following:

1. Enable "Sign in with Microsoft"
2. Fill in "Client ID" with "Application (client) ID" above
3. Fill in "Client secret" with the secret you get after creating a client secret for your app.
4. For "Tenant" field:
   * If single tenant (first option) is chosen, input the "Directory (tenant) ID"
   * If multi tenant (second option) is chosen, input the string "organizations"
   * If multi tenant and personal account (third option) is chosen, input the string "common"
5. **Save** the settings

🎉 Done! You have just added Azure Active Directory (Azure AD) Login to your apps!
