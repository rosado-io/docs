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

### mail.smtp

`mail.smtp` defines the SMTP credentials.

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

`jwt` defines the JWS of generated JWT.

```yaml
secrets:
- key: jwt
  data:
    keys:
    - kid: key_id
      kty: key_type
      # TODO(jws): Document JWS
```

### oidc

`oidc` defines the JWS to sign the generated ID tokens generated.

The format shares with [jwt](#jwt)

### csrf

`csrf` defines the symmetric key to generate CSRF token.

The format shares with [jwt](#jwt)

### webhook

`webhook` defines the symmetric key to sign webhook request body.

The format shares with [jwt](#jwt)
