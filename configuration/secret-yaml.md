---
description: The secret configuration authgear.secrets.yaml
---

# authgear.secrets.yaml

This is the configuration file containing various secrets used in Authgear.

## JSON Schema

The configuration file is validated against the following JSON Schema.

```json
{
  "$defs": {
    "CSRFKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "DatabaseCredentials": {
      "additionalProperties": false,
      "properties": {
        "database_schema": {
          "type": "string"
        },
        "database_url": {
          "type": "string"
        }
      },
      "required": [
        "database_url"
      ],
      "type": "object"
    },
    "JWK": {
      "properties": {
        "kid": {
          "type": "string"
        },
        "kty": {
          "type": "string"
        }
      },
      "required": [
        "kid",
        "kty"
      ],
      "type": "object"
    },
    "JWS": {
      "additionalProperties": false,
      "properties": {
        "keys": {
          "items": {
            "$ref": "#/$defs/JWK"
          },
          "minItems": 1,
          "type": "array"
        }
      },
      "required": [
        "keys"
      ],
      "type": "object"
    },
    "JWTKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "NexmoCredentials": {
      "additionalProperties": false,
      "properties": {
        "api_key": {
          "type": "string"
        },
        "api_secret": {
          "type": "string"
        }
      },
      "required": [
        "api_key",
        "api_secret"
      ],
      "type": "object"
    },
    "OAuthClientCredentials": {
      "additionalProperties": false,
      "properties": {
        "items": {
          "items": {
            "additionalProperties": false,
            "properties": {
              "alias": {
                "type": "string"
              },
              "client_secret": {
                "type": "string"
              }
            },
            "required": [
              "alias",
              "client_secret"
            ],
            "type": "object"
          },
          "type": "array"
        }
      },
      "required": [
        "items"
      ],
      "type": "object"
    },
    "OIDCKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "RedisCredentials": {
      "additionalProperties": false,
      "properties": {
        "db": {
          "type": "integer"
        },
        "host": {
          "type": "string"
        },
        "password": {
          "type": "string"
        },
        "port": {
          "type": "integer"
        },
        "sentinel": {
          "$ref": "#/$defs/RedisSentinelConfig"
        }
      },
      "type": "object"
    },
    "RedisSentinelConfig": {
      "additionalProperties": false,
      "properties": {
        "addrs": {
          "items": {
            "type": "string"
          },
          "type": "array"
        },
        "enabled": {
          "type": "boolean"
        },
        "master_name": {
          "type": "string"
        }
      },
      "type": "object"
    },
    "SMTPMode": {
      "enum": [
        "normal",
        "ssl"
      ],
      "type": "string"
    },
    "SMTPServerCredentials": {
      "additionalProperties": false,
      "properties": {
        "host": {
          "type": "string"
        },
        "mode": {
          "$ref": "#/$defs/SMTPMode"
        },
        "password": {
          "type": "string"
        },
        "port": {
          "maximum": 65535,
          "minimum": 1,
          "type": "integer"
        },
        "username": {
          "type": "string"
        }
      },
      "required": [
        "host",
        "port"
      ],
      "type": "object"
    },
    "SecretConfig": {
      "additionalProperties": false,
      "properties": {
        "secrets": {
          "items": {
            "$ref": "#/$defs/SecretItem"
          },
          "type": "array"
        }
      },
      "required": [
        "secrets"
      ],
      "type": "object"
    },
    "SecretItem": {
      "additionalProperties": false,
      "allOf": [
        {
          "if": {
            "properties": {
              "key": {
                "const": "csrf"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/CSRFKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "db"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/DatabaseCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "jwt"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/JWTKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "mail.smtp"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/SMTPServerCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "oidc"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/OIDCKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "redis"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/RedisCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sms.nexmo"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/NexmoCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sms.twilio"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/TwilioCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sso.oauth.client"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/OAuthClientCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "webhook"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/WebhookKeyMaterials"
              }
            }
          }
        }
      ],
      "properties": {
        "data": {},
        "key": {
          "$ref": "#/$defs/SecretKey"
        }
      },
      "required": [
        "key",
        "data"
      ],
      "type": "object"
    },
    "SecretKey": {
      "enum": [
        "csrf",
        "db",
        "jwt",
        "mail.smtp",
        "oidc",
        "redis",
        "sms.nexmo",
        "sms.twilio",
        "sso.oauth.client",
        "webhook"
      ],
      "type": "string"
    },
    "TwilioCredentials": {
      "additionalProperties": false,
      "properties": {
        "account_sid": {
          "type": "string"
        },
        "auth_token": {
          "type": "string"
        }
      },
      "required": [
        "account_sid",
        "auth_token"
      ],
      "type": "object"
    },
    "WebhookKeyMaterials": {
      "$ref": "#/$defs/JWS"
    }
  },
  "$ref": "#/$defs/SecretConfig"
}
```

## Structure

Secrets are placed under the key `secrets`. Each item has `key` and `data`. The valid values for `key` are listed below. `data` is key-specific.

```yaml
secrets:
- key: well-known-key
  data: key-specific-data
```

Note that **ALL** secrets are required.

### db

`db` defines the database credentials. Only PostgreSQL database is supported.

```yaml
secrets:
- key: db
  data:
    database_url: postgresql://
    database_scheme: app
```

### redis

`redis` defines the Redis credentials.

```yaml
secrets:
- key: redis
  data:
    host: localhost
    port: 6379
    password: password
    db: 0
```

### sso.oauth.client

`sso.oauth.client` defines the client secrets.

This is the place where you provide the client secrets of configured external OAuth providers.

```yaml
secrets:
- key: sso.oauth.client
  data:
    items:
    - alias: google
      client_secret: client_secret
    - alias: apple
      client_secret: private_key_in_pem_format
```

### mail.smtp

`mail.smtp` defines the SMTP credentials.

`mode` is either `ssl` or `normal`. Usually you do not need to set it and the mode is inferred from the port.

```yaml
secrets:
- key: mail.smtp
  data:
    host: localhost
    port: 587
    mode: ssl
    username: username
    password: password
```

### sms.twilio

`sms.twilio` defines the Twilio credentials.

```yaml
secrets:
- key: sms.twilio
  data:
    account_sid: account_sid
    auth_token: auth_token
```

### sms.nexmo

`sms.nexmo` defines the Nexmo credentials.

```yaml
secrets:
- key: sms.nexmo
  data:
    api_key: api_key
    api_secret: api_secret
```

### jwt

`jwt` defines the JWK to sign internal use, ephemeral JWT token.
It must be a octet key.

```yaml
secrets:
- key: jwt
  data:
    keys:
    - kid: key_id
      kty: oct
      k: key
```

### oidc

`oidc` defines the JWK to sign ID tokens.
It must be a RSA key.

```yaml
secrets:
- key: oidc
  data:
    keys:
    - kid: key_id
      kty: RSA
      # Other fields specific to RSA.
```

### csrf

`csrf` defines the symmetric key to generate CSRF token.
It must be a octet key.

The format shares with [jwt](#jwt)

### webhook

`webhook` defines the symmetric key to sign webhook request body.
It must be a octet key.

The format shares with [jwt](#jwt)
