---
description: Environment variable provides global configuration
---

# Environment Variable

## Environment Variable

Environment variable provides global configuration.

### PUBLIC\_LISTEN\_ADDR

This sets the listen address of the public server. The default is `0.0.0.0:3000`.

### INTERNAL\_LISTEN\_ADDR

This sets the listen address of the internal server. The default is `0.0.0.0:3001`.

### TRUST\_PROXY

This sets whether incoming HTTP headers such as `x-forwarded-host` can be trusted. If you deploy Authgear behind a reverse proxy capable of writing these headers, you should set the value to `true`. The default is `false`.

### DEV\_MODE

This sets whether Authgear should run in development mode. You should never need to set it. The default is `false`.

### LOG\_LEVEL

This sets the global log level. Valid values are `debug`, `info`, `warn` and `error`. The default is `warn`.

### CONFIG\_SOURCE\_TYPE

This sets the type of the configuration. The only supported value for now is `local`. The default is `local`, so you should never change it.

### CONFIG\_SOURCE\_APP\_CONFIG\_PATH

This sets the filepath of the app configuration. The default is `authgear.yaml` so `authgear.yaml` in the working directory is loaded.

### CONFIG\_SOURCE\_SECRET\_CONFIG\_PATH

This sets the filepath of the secret configuration. The default is `authgear.secrets.yaml` so `authgear.secrets.yaml` in the working directory is loaded.

### RESERVED\_NAME\_FILE\_PATH

This sets the filepath of the file containing reserved usernames. The default is `reserved_name.txt`. A default file is bundled along with the provided Docker image so you should only change it if you really want to override.

### STATIC\_ASSET\_SERVING\_ENABLED

This sets whether the bundled static asset should be served. Default is `true`. You should never modify it.

### STATIC\_ASSET\_DIR

This sets the filepath of the directory containing the bundled static asset. The default value of the provided Docker image does the right thing so you should never need to set it.

### STATIC\_ASSET\_URL\_PREFIX

This sets the URL prefix of the bundled static asset. The default value includes commit hash so it is cache-friendly.

## TL;DR

The only environment variable you should be aware of is [TRUST\_PROXY](env.md#trust_proxy).

