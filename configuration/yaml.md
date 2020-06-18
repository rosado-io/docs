---
description: The app configuration authgear.yaml
---

# authgear.yaml

This is the main configuration file affecting every aspect of Authgear.

## JSON Schema

The configuration file is validated against the following JSON Schema.

```json
{
    "$ref": "#/$defs/AppConfig",
    "$defs": {
        "AppConfig": {
            "properties": {
                "authentication": {
                    "$ref": "#/$defs/AuthenticationConfig"
                },
                "authenticator": {
                    "$ref": "#/$defs/AuthenticatorConfig"
                },
                "forgot_password": {
                    "$ref": "#/$defs/ForgotPasswordConfig"
                },
                "hook": {
                    "$ref": "#/$defs/HookConfig"
                },
                "http": {
                    "$ref": "#/$defs/HTTPConfig"
                },
                "id": {
                    "type": "string"
                },
                "identity": {
                    "$ref": "#/$defs/IdentityConfig"
                },
                "localization": {
                    "$ref": "#/$defs/LocalizationConfig"
                },
                "metadata": {
                    "$ref": "#/$defs/AppMetadata"
                },
                "oauth": {
                    "$ref": "#/$defs/OAuthConfig"
                },
                "session": {
                    "$ref": "#/$defs/SessionConfig"
                },
                "template": {
                    "$ref": "#/$defs/TemplateConfig"
                },
                "ui": {
                    "$ref": "#/$defs/UIConfig"
                },
                "welcome_message": {
                    "$ref": "#/$defs/WelcomeMessageConfig"
                }
            },
            "required": [
                "id"
            ],
            "type": "object"
        },
        "AppMetadata": {
            "patternProperties": {
                "^app_name(#.+)?$": {
                    "type": "string"
                },
                "^logo_uri(#.+)?$": {
                    "format": "uri",
                    "type": "string"
                }
            },
            "type": "object"
        },
        "AuthenticationConfig": {
            "properties": {
                "identities": {
                    "items": {
                        "$ref": "#/$defs/IdentityType"
                    },
                    "type": "array",
                    "uniqueItems": true
                },
                "primary_authenticators": {
                    "items": {
                        "$ref": "#/$defs/AuthenticatorType"
                    },
                    "type": "array",
                    "uniqueItems": true
                },
                "secondary_authentication_mode": {
                    "$ref": "#/$defs/SecondaryAuthenticationMode"
                },
                "secondary_authenticators": {
                    "items": {
                        "$ref": "#/$defs/AuthenticatorType"
                    },
                    "type": "array",
                    "uniqueItems": true
                }
            },
            "type": "object"
        },
        "AuthenticatorBearerTokenConfig": {
            "properties": {
                "expire_in_days": {
                    "$ref": "#/$defs/DurationDays"
                }
            },
            "type": "object"
        },
        "AuthenticatorConfig": {
            "properties": {
                "bearer_token": {
                    "$ref": "#/$defs/AuthenticatorBearerTokenConfig"
                },
                "oob_otp": {
                    "$ref": "#/$defs/AuthenticatorOOBConfig"
                },
                "password": {
                    "$ref": "#/$defs/AuthenticatorPasswordConfig"
                },
                "recovery_code": {
                    "$ref": "#/$defs/AuthenticatorRecoveryCodeConfig"
                },
                "totp": {
                    "$ref": "#/$defs/AuthenticatorTOTPConfig"
                }
            },
            "type": "object"
        },
        "AuthenticatorOOBConfig": {
            "properties": {
                "email": {
                    "$ref": "#/$defs/AuthenticatorOOBEmailConfig"
                },
                "sms": {
                    "$ref": "#/$defs/AuthenticatorOOBSMSConfig"
                }
            },
            "type": "object"
        },
        "AuthenticatorOOBEmailConfig": {
            "properties": {
                "maximum": {
                    "type": "integer"
                },
                "message": {
                    "$ref": "#/$defs/EmailMessageConfig"
                }
            },
            "type": "object"
        },
        "AuthenticatorOOBSMSConfig": {
            "properties": {
                "maximum": {
                    "type": "integer"
                },
                "message": {
                    "$ref": "#/$defs/SMSMessageConfig"
                }
            },
            "type": "object"
        },
        "AuthenticatorPasswordConfig": {
            "properties": {
                "policy": {
                    "$ref": "#/$defs/PasswordPolicyConfig"
                }
            },
            "type": "object"
        },
        "AuthenticatorRecoveryCodeConfig": {
            "properties": {
                "count": {
                    "type": "integer"
                },
                "list_enabled": {
                    "type": "integer"
                }
            },
            "type": "object"
        },
        "AuthenticatorTOTPConfig": {
            "properties": {
                "maximum": {
                    "type": "integer"
                }
            },
            "type": "object"
        },
        "AuthenticatorType": {
            "enum": [
                "password",
                "totp",
                "oob_otp",
                "bearer_token"
            ],
            "type": "string"
        },
        "DurationDays": {
            "type": "integer"
        },
        "DurationSeconds": {
            "type": "integer"
        },
        "EmailMessageConfig": {
            "patternProperties": {
                "^reply_to(#.+)?$": {
                    "format": "email-name-addr",
                    "type": "string"
                },
                "^sender(#.+)?$": {
                    "format": "email-name-addr",
                    "type": "string"
                },
                "^subject(#.+)?$": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "ForgotPasswordConfig": {
            "properties": {
                "email_message": {
                    "$ref": "#/$defs/EmailMessageConfig"
                },
                "reset_code_expiry_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                },
                "sms_message": {
                    "$ref": "#/$defs/SMSMessageConfig"
                }
            },
            "type": "object"
        },
        "HTTPConfig": {
            "properties": {
                "allowed_origins": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                },
                "hosts": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "HookConfig": {
            "properties": {
                "handlers": {
                    "items": {
                        "$ref": "HookHandlerConfig"
                    },
                    "type": "array"
                },
                "sync_hook_timeout_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                },
                "sync_hook_total_timeout_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                }
            },
            "type": "object"
        },
        "HookHandlerConfig": {
            "properties": {
                "event": {
                    "type": "string"
                },
                "url": {
                    "format": "uri",
                    "type": "string"
                }
            },
            "required": [
                "event",
                "url"
            ],
            "type": "object"
        },
        "IdentityConfig": {
            "properties": {
                "login_id": {
                    "$ref": "#/$defs/LoginIDConfig"
                },
                "sso": {
                    "$ref": "#/$defs/IdentityConflictConfig"
                }
            },
            "type": "object"
        },
        "IdentityConflictConfig": {
            "properties": {
                "promotion": {
                    "$ref": "#/$defs/PromotionConflictBehavior"
                }
            },
            "type": "object"
        },
        "IdentityType": {
            "enum": [
                "login_id",
                "oauth",
                "anonymous"
            ],
            "type": "string"
        },
        "LocalizationConfig": {
            "properties": {
                "fallback_language": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "LoginIDConfig": {
            "properties": {
                "keys": {
                    "items": {
                        "$ref": "#/$defs/LoginIDKeyConfig"
                    },
                    "type": "array"
                },
                "types": {
                    "$ref": "#/$defs/LoginIDTypesConfig"
                }
            },
            "type": "object"
        },
        "LoginIDEmailConfig": {
            "properties": {
                "block_plus_sign": {
                    "type": "boolean"
                },
                "case_sensitive": {
                    "type": "boolean"
                },
                "ignore_dot_sign": {
                    "type": "boolean"
                }
            },
            "type": "object"
        },
        "LoginIDKeyConfig": {
            "properties": {
                "key": {
                    "type": "string"
                },
                "maximum": {
                    "type": "integer"
                },
                "type": {
                    "$ref": "#/$defs/LoginIDKeyType"
                }
            },
            "required": [
                "key",
                "type"
            ],
            "type": "object"
        },
        "LoginIDKeyType": {
            "enum": [
                "raw",
                "email",
                "phone",
                "username"
            ],
            "type": "string"
        },
        "LoginIDTypesConfig": {
            "properties": {
                "email": {
                    "$ref": "#/$defs/LoginIDEmailConfig"
                },
                "username": {
                    "$ref": "#/$defs/LoginIDUsernameConfig"
                }
            },
            "type": "object"
        },
        "LoginIDUsernameConfig": {
            "properties": {
                "ascii_only": {
                    "type": "boolean"
                },
                "block_reserved_usernames": {
                    "type": "boolean"
                },
                "case_sensitive": {
                    "type": "boolean"
                },
                "excluded_keywords": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "MessagingConfig": {
            "properties": {
                "default_email_message": {
                    "$ref": "#/$defs/EmailMessageConfig"
                },
                "default_sms_message": {
                    "$ref": "#/$defs/SMSMessageConfig"
                },
                "sms_provider": {
                    "$ref": "#/$defs/SMSProvider"
                }
            },
            "type": "object"
        },
        "OAuthClientConfig": {
            "properties": {
                "access_token_lifetime_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                },
                "client_id": {
                    "type": "string"
                },
                "client_uri": {
                    "format": "uri",
                    "type": "string"
                },
                "grant_types": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                },
                "post_logout_redirect_uris": {
                    "items": {
                        "format": "uri",
                        "type": "string"
                    },
                    "type": "array"
                },
                "redirect_uris": {
                    "items": {
                        "format": "uri",
                        "type": "string"
                    },
                    "minItems": 1,
                    "type": "array"
                },
                "refresh_token_lifetime_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                },
                "response_types": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                }
            },
            "required": [
                "client_id",
                "redirect_uris"
            ],
            "type": "object"
        },
        "OAuthConfig": {
            "properties": {
                "clients": {
                    "items": {
                        "$ref": "#/$defs/OAuthClientConfig"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "OAuthSSOProviderConfig": {
            "properties": {
                "alias": {
                    "type": "string"
                },
                "client_id": {
                    "type": "string"
                },
                "key_id": {
                    "type": "string"
                },
                "team_id": {
                    "type": "string"
                },
                "tenant": {
                    "type": "string"
                },
                "type": {
                    "$ref": "#/$defs/OAuthSSOProviderType"
                }
            },
            "required": [
                "type",
                "client_id"
            ],
            "type": "object"
        },
        "OAuthSSOProviderType": {
            "enum": [
                "google",
                "facebook",
                "linkedin",
                "azureadv2",
                "apple"
            ],
            "type": "string"
        },
        "PasswordPolicyConfig": {
            "properties": {
                "digit_required": {
                    "type": "boolean"
                },
                "excluded_keywords": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                },
                "history_days": {
                    "$ref": "#/$defs/DurationDays"
                },
                "history_size": {
                    "type": "integer"
                },
                "lowercase_required": {
                    "type": "boolean"
                },
                "min_length": {
                    "type": "integer"
                },
                "minimum_guessable_level": {
                    "type": "integer"
                },
                "symbol_required": {
                    "type": "boolean"
                },
                "uppercase_required": {
                    "type": "boolean"
                }
            },
            "type": "object"
        },
        "PromotionConflictBehavior": {
            "enum": [
                "error",
                "login"
            ],
            "type": "string"
        },
        "SMSMessageConfig": {
            "properties": {
                "^sender(#.+)?$": {
                    "format": "phone",
                    "type": "string"
                }
            },
            "type": "object"
        },
        "SMSProvider": {
            "enum": [
                "nexmo",
                "twilio"
            ],
            "type": "string"
        },
        "SSOConfig": {
            "properties": {
                "oauth_providers": {
                    "items": {
                        "$ref": "#/$defs/OAuthSSOProviderConfig"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "SecondaryAuthenticationMode": {
            "enum": [
                "if_requested",
                "if_exists",
                "required"
            ],
            "type": "string"
        },
        "SessionConfig": {
            "properties": {
                "cookie_domain": {
                    "type": "string"
                },
                "cookie_non_persistent": {
                    "type": "boolean"
                },
                "idle_timeout_enabled": {
                    "type": "boolean"
                },
                "idle_timeout_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                },
                "lifetime_seconds": {
                    "$ref": "#/$defs/DurationSeconds"
                }
            },
            "type": "object"
        },
        "TemplateConfig": {
            "properties": {
                "items": {
                    "items": {
                        "$ref": "#/$defs/TemplateItem"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "TemplateItem": {
            "properties": {
                "key": {
                    "type": "string"
                },
                "language_tag": {
                    "type": "string"
                },
                "type": {
                    "$ref": "#/$defs/TemplateItemType"
                },
                "uri": {
                    "type": "string"
                }
            },
            "required": [
                "type",
                "uri"
            ],
            "type": "object"
        },
        "TemplateItemType": {
            "type": "string"
        },
        "UIConfig": {
            "properties": {
                "country_calling_code": {
                    "$ref": "#/$defs/UICountryCallingCodeConfig"
                },
                "custom_css": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "UICountryCallingCodeConfig": {
            "properties": {
                "default": {
                    "type": "string"
                },
                "values": {
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                }
            },
            "type": "object"
        },
        "WelcomeMessageConfig": {
            "properties": {
                "destination": {
                    "$ref": "#/$defs/WelcomeMessageDestination"
                },
                "email_message": {
                    "$ref": "#/$defs/EmailMessageConfig"
                },
                "enabled": {
                    "type": "boolean"
                }
            },
            "type": "object"
        },
        "WelcomeMessageDestination": {
            "enum": [
                "first",
                "all"
            ],
            "type": "string"
        }
    }
}
```

## Annotated example

```yaml
# The ID of this instance of Authgear.
id: myapp
# Configure the authentication behavior.
authentication:
  # Determine which identities are enabled.
  # By default "login_id" and "oauth" are enabled.
  identities:
  - login_id
  - oauth
  - anonymous
  # Determine which authenticators can be used as primary authenticator.
  # By default only "password" is enabled.
  primary_authenticators:
  - password
  - totp
  - oob_otp
  # Determine which authenticators can be used as secondary authenticator.
  # By default totp, oob_otp and bearer_token are enabled.
  secondary_authenticators:
  - totp
  - oob_otp
  - bearer_token
  # Configure the MFA behavior.
  #
  # if_exists: The user can add secondary authenticators.
  # If the user has at least one secondary authenticator, then MFA must be performed.
  #
  # required: The user must add secondary authenticators.
  # The user must perform MFA during authentication.
  #
  # if_requested: MFA is entirely optional even the user has at least one secondary authenticator.
  #
  # Default is "if_exists"
  secondary_authentication_mode: if_exists
# Configure different authenticator behavior.
authenticator:
  # Configure Bearer Token Authenticator.
  # Bearer token can be generated during MFA.
  # It is used to skip MFA on the device for future authentication.
  bearer_token:
    # Determine how long the bearer token is valid.
    expire_in_days: 30
  # Configure OOB-OTP Authenticator.
  oob_otp:
    email:
      # the maximum number of the authenticator the user can have.
      # default is 1.
      maximum: 1
      # message is EmailMessageConfig
      message:
        reply_to: no-reply@example.com
        sender: System <system@example.com>
        subject: Verify your email
    sms:
      # the maximum number of the authenticator the user can have.
      # default is 1.
      maximum: 1
      # message is SMSMessageConfig
      message:
       sender: +85299887766
  # Configure Password Authenticator
  password:
    # Configure password policy
    # All policies are turned off by default.
    policy:
      # Set the minimum length of new password.
      min_length: 10
      # Require new password to have at least 1 digit.
      digit_required: true
      # Require new password to have at least 1 lowercase ASCII character.
      lowercase_required: true
      # Require new password to have at least 1 uppercase ASCII character.
      uppercase_required: true
      # Require new password to have at least 1 symbol character.
      symbol_required: true
      # Disallow password containing the given keywords.
      excluded_keywords:
      - secret
      - admin
      - password
      # Require strong password.
      # The strength of the password is calculated with https://github.com/dropbox/zxcvbn
      # 1 is the weakest level and 5 is the strongest level.
      minimum_guessable_level: 5
      # Determine how long password history is kept.
      history_days: 90
      # Determine how many password history is kept.
      history_size: 10
  # Configure Recovery Code Authenticator
  recovery_code:
    # The number of recovery codes. Default is 16.
    count: 16
    # Whether the user can list the recovery codes again. Default is false.
    list_enabled: false
  # Configure TOTP Authenticator
  totp:
    # the maximum number of the authenticator the user can have.
    # default is 1.
    maximum: 1
# Configure forgot password behavior
forgot_password:
  # How long the reset code remains valid. The default is 1200. That is 20 minutes.
  reset_code_expiry_seconds: 1200
  # email_message is EmailMessageConfig
  email_message: {}
  # sms_message is SMSMessageConfig
  sms_message: {}
# Configure webhook
hook:
  # How long a single handler can proceed the webhook event before timeout.
  # Default is 5.
  sync_hook_timeout_seconds: 5
  # How long all handlers can proceed the webhook event before timeout.
  # Default is 10.
  sync_hook_total_timeout_seconds: 10
  # Define the list of webhook handlers
  handlers:
    # The event name.
  - event: before_user_create
    # The endpoint of the webhook handler.
    url: https://api.example.com/hook/before_user_create
http:
  # The allowed origin for the HTTP header access-control-allow-origin
  # Default is empty list.
  allowed_origins:
  - https://trusted-third-party-server.com
  # The expected host
  # Default is empty list.
  hosts:
  - https://accounts.myapp.com
# Configure different identity behavior.
identity:
  # Configure Login ID Identity.
  login_id:
    # Defines the set of accepted login IDs.
    # The configuration shown here is the default.
    # So by default the user can have
    #
    # At most 1 email
    # At most 1 phone
    # At most 1 username
    #
    # If you do not want the defaults, define keys yourselves.
    keys:
      # Define the key of the login ID.
    - key: email
      # Define the type of login ID.
      # Valid values are "email" "phone" "username" and "raw"
      type: email
      # How many login ID the user can have.
      # Default is 1.
      maximum: 1
    - key: phone
      type: phone
    - key: username
      type: username
    # Configure the characteristics of some login IDs.
    types:
      # Configure Email Login ID Identity.
      email:
        # Whether + sign should be disallowed in the local part. Default is false.
        block_plus_sign: false
        # Whether the email should be treated case sensitively. Default is false.
        case_sensitive: false
        # Whether . sign should be ignored. Default is false.
        ignore_dot_sign: false
      # Configure Username Login ID Identity.
      username:
        # Whether the username can only contain `-a-zA-Z0-9_.`. Default is true.
        ascii_only: true
        # Whether reserved usernames are blocked. Default is true.
        block_reserved_usernames: true
        # Whether the username should be treated case sensitively. Default is false.
        case_sensitive: false
        # Define a list of banned usernames. Default is empty list.
        excluded_keywords:
        - admin
  # Configure OAuth Identity.
  oauth:
    # Configure external OAuth identity providers.
    providers:
    # Denote the type of the identity provider.
    # Valid values are "google", "apple", "facebook", "azureadv2", "linkedin"
    - type: google
      # Client ID and client secret are the credentials you obtain from the specific provider.
      # Please refer to the documentation of the provider.
      client_id: google_client_id
      client_secret: google_client_secret
    - type: apple
      # The client ID for Apple is the services ID.
      client_id: apple_services_id
      # The client secret for Apple is the PEM format of the private key.
      client_secret: private_key
      # The key ID of the private key.
      key_id: key_id
      # The team ID of your Apple Developer Account.
      team_id: team_id
    - type: azureadv2
      client_id: client_id
      client_secret: client_secret
      # Tenant is either the special value "common", the special value "organizations" or
      # the ID of a Azure AD tenant.
      #
      # Note that when you create the client in Azure Portal,
      # you have to choose which tenant the client intends to interact with.
      #
      # If you wish to allow any microsoft accounts such as
      #
      # - hotmail
      # - Xbox
      # - Outlook
      #
      # to login, then the value must be "common".
      #
      # If you wish to allow any user in any Azure ADs in your Azure account,
      # then the value must be "organizations".
      #
      # Otherwise the value must be the ID of a Azure AD tenant.
      # In this case, only user in that Azure AD can login.
      tenant: common
# Configure localization.
localization:
  # The fallback language when none of the supported languages match the preferred languages.
  # Default is en.
  fallback_language: en
# Configure the metadata of Authgear.
metadata:
  # Set the user-facing app name.
  # It is shown in the web UI.
  app_name: My App
  app_name#ja-JP: アプリ
  # Set the logo URI in the web UI.
  logo_uri: https://static.example.com/logo.png
# Configure OAuth.
oauth:
  # Define the list of known OAuth 2 clients.
  clients:
    # The OAuth 2 client ID.
    # A reasonably long secure random string is recommended.
  - client_id: client_id
    # Define a list of allowed redirect URIs.
    # According to OAuth 2 spec, exact match is used.
    # So "http://example.com" does not match "http://example.com/"
    redirect_uris:
    - "com.myapp://host/path"
    # Which grant types are allowed in the token endpoint.
    # "authorization_code" and "refresh_token" must be included for OAuth 2 authorization code flow.
    grant_types:
    - authorization_code
    - refresh_token
    # What response the authorization endpoint can return.
    # "code" must be included for OAuth 2 authorization code flow.
    response_types:
    - code
    # Define a list of allowed post logout redirect URIs.
    # Same as redirect_uris, exact match is used.
    post_logout_redirect_uris:
    - "https://www.myapp.com/post_logout"
    # The lifetime of the access token. Default is 1800.
    access_token_lifetime_seconds: 1800
    # The lifetime of the refresh token. Default is 86400.
    refresh_token_lifetime_seconds: 86400
# Configure session.
session:
  # Explicitly set the cookie domain. Default is eTLD+1.
  cookie_domain: https://accounts.myapp.com
  # Whether the cookie is session cookie. Default is false.
  cookie_non_persistent: false
  # Whether the session becomes invalid after idling.
  idle_timeout_enabled: false
  # How long before the session timeout.
  idle_timeout_seconds: 300
  # The lifetime of the session. Default is 86400.
  lifetime_seconds: 86400
# Configure template.
template:
  # Override default templates or provide additional translation files.
  items:
    # The type of template.
  - type: auth_ui_translation.json
    # The language of the template file.
    language_tag: ja-JP
    # The URI to load the template.
    # Only file scheme is supported.
    uri: file:///app/templates/auth_ui_transation.ja.json
ui:
  # Define a custom inline stylesheet to be injected in every pages of the UI.
  custom_css: |
    .a { color: red; }
  country_calling_code:
    # The default selected value of the country calling code.
    # Default is the first item in values.
    default: 852
    # The list of country calling code to show in the phone number input widget.
    values:
    - 852
# Configure welcome message.
welcome_message:
  # Whether to send the welcome message.
  # Default is false.
  enabled: false
  # Whether to send the welcome message to all addresses or first address.
  # Valid values are first and all.
  # Default is first.
  destination: first
  # email_message is EmailMessageConfig.
  email_message: {}
```
