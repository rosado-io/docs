---
description: Environment variable provides global configuration
---

# Environment Variable

## Environment Variable

Environment variable provides global configuration.

### TRUST\_PROXY

This sets whether incoming HTTP headers such as `x-forwarded-host` can be trusted. If you deploy Authgear behind a reverse proxy capable of writing these headers, you should set the value to `true`. The default is `false`.

### DEV\_MODE

This sets whether Authgear should run in development mode. You should never need to set it. The default is `false`.

When development mode is enabled:

* TLS certificate is required, to enable secure cookies.
* All `Host` header values are allowed.
* External message sending \(SMS/Email\) is disabled; messages to send are logged instead.

### LOG\_LEVEL

This sets the global log level. Valid values are `debug`, `info`, `warn` and `error`. The default is `warn`.

### STATIC\_ASSET\_URL\_PREFIX

This sets the URL prefix of the bundled static asset. The default value includes commit hash so it is cache-friendly.

### SENTRY\_DSN

The sets the Sentry DSN, where errors/logs are reported to.

### MAIN\_LISTEN\_ADDR

This sets the listen address of the main server. The default is `0.0.0.0:3000`.

### RESOLVER\_LISTEN\_ADDR

This sets the listen address of the resolver server. The default is `0.0.0.0:3001`.

### ADMIN\_LISTEN\_ADDR

This sets the listen address of the Admin API server. The default is `0.0.0.0:3002`.

### ADMIN\_API\_AUTH

This sets the authorization mode of the Admin API. Valid values are `jwt` and `none`. The default is `jwt`.

When the value is `jwt`, all requests to the Admin API must bear a valid JWT.

When the value is `none`, no authorization is needed. You must **NOT** use `none` in production unless you know the implied consequences.

### CONFIG\_SOURCE\_TYPE

This sets the type of the configuration. The only supported value for now is `local`. The default is `local`, so you should never change it.

### CONFIG\_SOURCE\_APP\_CONFIG\_PATH

This sets the filepath of the app configuration. The default is `authgear.yaml` so `authgear.yaml` in the working directory is loaded.

### CONFIG\_SOURCE\_SECRET\_CONFIG\_PATH

This sets the filepath of the secret configuration. The default is `authgear.secrets.yaml` so `authgear.secrets.yaml` in the working directory is loaded.

### DEFAULT\_TEMPLATE\_DIRECTORY

This sets the path to directory containing the default template files. The default is `templates`. Default template files are bundled along with the provided Docker image so you should only change it if you really want to override.

### RESERVED\_NAME\_FILE\_PATH

This sets the filepath of the file containing reserved usernames. The default is `reserved_name.txt`. A default file is bundled along with the provided Docker image so you should only change it if you really want to override.

### STATIC\_ASSET\_SERVING\_ENABLED

This sets whether the bundled static asset should be served. Default is `true`. You should never modify it.

### STATIC\_ASSET\_DIR

This sets the filepath of the directory containing the bundled static asset. The default value of the provided Docker image does the right thing so you should never need to set it.

## TL;DR

The only environment variable you should be aware of is [TRUST\_PROXY](env.md#trust_proxy).
