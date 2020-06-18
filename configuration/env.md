---
description: Environment variable provides global configuration
---

# Environment Variable

Environment variable provides global configuration.

## LISTEN_ADDR

Sets which interface and port should be listened. Default is `0.0.0.0:3000`.

## TRUST_PROXY

Sets whether incoming HTTP headers such as `x-forwarded-host` can be trusted. If you deploy Authgear behind a reverse proxy capable of writing these headers, you should set the value to `true`. Default is `false`.

## DEV_MODE

Sets whether Authgear should run in development mode. You should never need to set it. Default is `false`.

## CONFIG_SOURCE_TYPE

Sets the type of the configuration. The only supported value for now is `local`. Default is `local`. So you should never change it.

## CONFIG_SOURCE_APP_CONFIG_PATH

Sets the filepath of the app configuration. Default is `authgear.yaml` so `authgear.yaml` in the working directory is loaded.

## CONFIG_SOURCE_SECRET_CONFIG_PATH

Sets the filepath of the secret configuration. Default is `authgear.secret.yaml` so `authgear.secret.yaml` in the working directory is loaded.

## RESERVED_NAME_FILE_PATH

Sets the filepath of the file containing reserved usernames. Default is `reserved_name.txt`. A default file is bundled along the provided Docker image so you should only change it if you really want to override.

## STATIC_ASSET_SERVING_ENABLED

Sets whether the bundled static asset should be served. Default is `true`. You should never modify it.

## STATIC_ASSET_DIR

Sets the filepath of the directory containing the bundled static asset. The default value of the provided Docker image does the right thing so you should never need to set it.

## STATIC_ASSET_URL_PREFIX

Sets the URL prefix of the bundled static asset. The default value includes commit hash so it is cache-friendly.

# TL;DR

The only environment variable you should be aware of is [TRUST_PROXY](#trust_proxy).
