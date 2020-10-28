# Social Sign-on Integration

## Sign in with Apple

#### Prerequisite

To configure "Sign in with Apple" for Authgear, you will need to fulfill the following:

1. Register an Apple Developer Account. Apple Enterprise Account does not support "Sign in with Apple"
2. Register your own domain.
3. Your domain must be able to send and receive emails.
4. Set up [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework)(SPF) for your domain.
5. Set up [DomainKeys Identified Mail](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)(DKIM) for your domain.
6. Create an "App ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose app IDs, enable "Sign in with Apple" enabled.
7. Create a "Services ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose service IDs, enable "Sign in with Apple".
8. Click "Configure" the Next to "Sign in with Apple". In "Primary App ID" field, select app ID created above.
9. Fill in and verify the domain created above, add `{Authgear endpoint}/sso/oauth2/callback/{alias}` to "Return URLs" (`alias` can be configured in Authgear portal)
10. Create a "Key" following [this guide](https://help.apple.com/developer-account/#/devcdfbb56a3) with "Sign in with Apple" enabled. Click "Configure" next to "Sign in with Apple" and select "Primary App ID" with app ID created above. Keep the private key safe, you need to provide this later.

#### Configure "Sign in with Apple" through portal

In portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with Apple"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID" field with "Service ID" above
4. In Apple Developer Portal, view key information of the "Key" created above
5. Jot down the "Key ID" and download the key text file (`.p8` file)
6. Copy the content in the key file to "Client Secret" text area in Authgear portal
7. Fill in "Key ID" field using "Key ID" from step 4
8. In Apple Developer Portal, click username on top right corner, click "View Membership"
9. Find the "Team ID" from "Membership Information", fill in "Team ID" field in Authgear portal
10. Scroll to bottom, and click save

### Google

#### Prerequisite

To configure Google OAuth client for Authgear, you will need to create a OAuth client on Google Cloud Platform.

1. Create a project on Google Cloud Platform through [console](https://console.cloud.google.com/)
2. Select APIs & services -> Credentials
3. Click "Create Credentials" and choose OAuth client ID and follow the instructions
4. Add `{Authgear endpoint}/sso/oauth2/callback/{alias}` to "redirect URIs" (`alias` can be configured in Authgear portal)
5. After creating a client ID, you will find the client ID in "OAuth 2.0 Client IDs" section in "Credentials" page.

You can find more details in [official Google Cloud Platform doc](https://support.google.com/cloud/answer/6158849)

#### Configure Sign in with Google through portal

After creating an OAuth client, click the name of OAuth client to view the details (see image below)

![gcp-oauth-client-details](../assets/gcp-download-oauth-client-details.png)

You will need the follwing fields:

1. "Client ID"
2. "Client secret"

In portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with Google"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in the client ID field and client secret field
4. Scroll to bottom and click save

### Facebook

#### Prerequisite

To configure "Login with Facebook" for Authgear, you will need the following

1. A [Facebook Developer Account](https://developers.facebook.com/apps/)
2. A [registered Facebook App](https://developers.facebook.com/docs/apps#register)
3. Go to app dashboard, click plus button next to "Product" in sidebar, add "Facebook Login".
4. Then go to "Settings" of "Facebook Login", add `{Authgear domain}/sso/oauth2/callback/{alias}` to "Valud OAuth Redirect URIs" (`alias` can be configured in Authgear portal)

#### Configure Login with Facebook through portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Login with Facebook"
2. In app dashboard of your app in Facebook developer portal, click "Settings" in sidebar and click "Basic"
3. In Authgear portal, fill in "Alias" with alias used in redirect URI
4. Fill in "Client ID" with "App ID"
5. Fill in "Client Secret" with "App Secret"
6. Scroll to bottom and click save

### Linkedin

#### Prerequisite

1. Create an app [here](https://www.linkedin.com/developers/)
2. In "Products" section, choose "Sign In with LinkedIn"
3. In details page of the created app, click "Auth" tab
4. Take notes of "Client ID" and "Client Secret", add `{Authgear domain}/sso/oauth2/callback/{alias}` to "Redirect URLs" in "OAuth 2.0 settings" section (`alias` can be configured in Authgear portal)

#### Configure Sign in with LinkedIn through portal

In Authgear portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with LinkedIn"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID" and "Client Secret"
4. Scroll down and click save

### Azure Active Directory

#### Prerequisite

1. Create an Azure Active Directory (Azure AD) account [here](https://azure.microsoft.com/free)
2. Setup a tenant by completing [Quickstart: Set up a tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-create-new-tenant)
3. Register an application by completing [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
4. Choose "Supported account type", the following options are supported:

   - Accounts in this organizational directory only (Contoso AD (dev) only - Single tenant)
   - Accounts in this organizational directory (Any Azure AD directory - Multitenant)
   - Accounts in this organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)

   "Personal Microsoft accounts only" is not supported yet. Remember the account type chosen as this affects the configuration on Authgear portal

5. Configure "Redirect URI" with `{Authgear domain}/sso/oauth2/callback/{alias}` (`alias` can be configured in Authgear portal)
6. Follow [this](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-a-client-secret) section to add a client secret. Remember to record the secret value when you add the client secret, as it will not be displayed again. This will be needed for configure OAuth client in Authgear.

#### Configure Sign in with Microsoft through portal

In Azure portal, go to the details page of your app, you can find:

1. Application (client) ID
2. Directory (tenant) ID

Then in Authgear portal, go to "Single-Sign On" page, and do the following:

1. Enable "Sign in with Microsoft"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID" with "Application (client) ID" above
4. Fill in "Client secret" with the secret you get after creating client secret for your app.
5. For "Tenant" field:
   - If single tenant (first option) is chosen, input the "Directory (tenant) ID"
   - If multi tenant (second option) is chosen, input the string "organizations"
   - If multi tenant and personal account (third option) is chosen, input the string "common"
6. Scroll down and click save

### Notes

- "alias" is used as identifier of OAuth provider
- Redirect URI has the form "/sso/oauth2/callback/:alias"
