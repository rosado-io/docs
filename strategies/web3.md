# Ethereum & NFT

Sign In With Ethereum(also known as SIWE) introduces a new way of authentication and identification of a user using their crypto-wallet addresses. Authgear saves you from all the complex setups and brings you the technology of the future with just a simple click.

#### Platform support

The Sign In With Ethereum standard is built on the [EIP-4361](https://eips.ethereum.org/EIPS/eip-4361) specification, which is typically performed on Web3 providers.

To make this as simple as possible, the following mainstream comsumer-facing web3 providers are selected to be the gateway connecting between your application and users.

* [MetaMask](https://metamask.io/)

These providers are typically supported by browsers with WebExtensions API, namely Chrome, Firefox, Edge and Brave.

#### NFT

To assist you in building the greatest NFT-gated application ever, Authgear adds extra information to the user info that indicates whether the user owns NFT/NFTs of your selected NFT collection.

At the current stage, the following token types are supported:

* [ERC-721](https://eips.ethereum.org/EIPS/eip-721)

**UserInfo**

```json
{
  "x_web3": {
    "accounts": [
      {
        "account_identifier": {
          "address": "0xec7f0e0c2b7a356b5271d13e75004705977fd010"
        },
        "network_identifier": {
          "blockchain": "ethereum",
          "network": "1"
        },
        "nfts": [
          {
            "contract": {
              "name": "ExampleCollection",
              "address": "0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85"
            },
            "balance": 1,
            "tokens": [
              {
                "token_id": 1,
                "transaction_identifier": {
                  "hash": "0x1a2bed0813d524955926eb190018d9d8738836265b352e1c43dc2d5762f9c20B"
                },
                "block_identifier": {
                  "index": 12961059,
                  "timestamp": "2022-09-01T08:17:50Z"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## Add Sign In With Ethereum to your apps with Authgear

To enable "Sign In With Ethereum" to your apps and websites:

1. In your project portal, go to "**Authentication > Ethereum & NFT**"
2. Turn on the "**Login With Ethereum**" toggle
3. Select your favourite blockchain network from the "**Network**" dropdown below
4. Press "Save" and "Confirm" and :tada: now your app supports sign in with ethereum!

{% hint style="info" %}
To prevent Web2 and Web3 identities being mixed together, Authgear has made it so that enabling "Sign In With Ethereum" would disable all your previously enabled identities and primary authenticators.

To go back to using Web2 authentication methods, you will have to disable "Sign In With Ethereum" and reconfigure your identities and authenticators.
{% endhint %}

## Add NFT collections to your apps with Authgear

To populate user info with NFT tokens from the collections of your choice:

1. In your project portal, go to "**Authentication > Ethereum & NFT**"
2. Ensure "**Login With Ethereum**" toggle is checked
3. Select the blockchain network from the "**Network**" for where your NFT collection is based on
4. In the "**NFT Collections**" section, press "**Add Collection**", this will add an extra text field to the section
5. Enter the contract address of the NFT collections in the "**Contract Address**" text field
6. Press "Add" and :tada: your app is now monitoring the NFT collection!

{% hint style="info" %}
It will take a few minutes for Authgear to index the NFT tokens for your collections. You may see the synchronization progress from the "**Detail**" modal that can be opened from the collection list item. Once the "**Block Height**" of the collection reaches the current height of the blockchain, it is fully synchronized and indexed. 
{% endhint %}