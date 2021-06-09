# Apple

## Prerequisite

To configure "Sign in with Apple" for Authgear, you will need to fulfil the following:

1. Register an Apple Developer Account. Apple Enterprise Account does not support "Sign in with Apple"
2. Register your own domain.
3. Your domain must be able to send and receive emails.
4. Set up [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework)\(SPF\) for your domain.
5. Set up [DomainKeys Identified Mail](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)\(DKIM\) for your domain.
6. Create an "App ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose app IDs, enable "Sign in with Apple" enabled.
7. Create a "Services ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose service IDs, enable "Sign in with Apple".
8. Click "Configure" the Next to "Sign in with Apple". In "Primary App ID" field, select app ID created above.
9. Fill in and verify the domain created above, add `https://{your domain}/sso/oauth2/callback/{alias}` to "Return URLs" \(`alias` can be configured in Authgear portal\)
10. Create a "Key" following [this guide](https://help.apple.com/developer-account/#/devcdfbb56a3) with "Sign in with Apple" enabled. Click "Configure" next to "Sign in with Apple" and select "Primary App ID" with app ID created above. Keep the private key safe, you need to provide this later.

## Configure "Sign in with Apple" through the portal

In the portal, go to "Single-Sign On" page, then do the following:

1. Enable "Sign in with Apple"
2. Fill in "Alias" with alias used in redirect URI
3. Fill in "Client ID" field with "Service ID" above
4. In Apple Developer Portal, view key information of the "Key" created above
5. Jot down the "Key ID" and download the key text file \(`.p8` file\)
6. Copy the content in the key file to "Client Secret" text area in Authgear portal
7. Fill in "Key ID" field using "Key ID" from step 4
8. In Apple Developer Portal, click username on the top right corner, click "View Membership"
9. Find the "Team ID" from "Membership Information", fill in "Team ID" field in Authgear portal
10. Scroll to the bottom, and click save

## Notes

* "alias" is used as the identifier of OAuth provider
* Redirect URI has the form "/sso/oauth2/callback/:alias"

