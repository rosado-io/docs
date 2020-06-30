---
description: The secret configuration authgear.secrets.yaml
---

# authgear.secrets.yaml

This is the configuration file containing various secrets used in Authgear.

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
