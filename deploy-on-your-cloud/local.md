---
description: How to run locally with Docker.
---

# Running locally with Docker

Authgear is available as a Docker image. It depends on PostgreSQL and Redis. To run it locally, the simplest way is to use docker-compose.

## Create the project directory

Let's get started with creating a new directory.

```bash
mkdir myapp
cd myapp
```

## Create authgear.yaml and authgear.secrets.yaml

First, we need to create authgear.yaml and authgear.secrets.yaml. Authgear itself is a CLI program capable of generating a minimal configuration file.

Run the following command to generate a minimal authgear.yaml:

```bash
docker run --rm -it -w "/work" -v "$PWD:/work" quay.io/theauthgear/authgear-server authgear init config
```

authgear.yaml is generated in your working directory.

Run the following command to generate a minimal authgear.secrets.yaml:

```bash
docker run --rm -it -w "/work" -v "$PWD:/work" quay.io/theauthgear/authgear-server authgear init secrets
```

authgear.secrets.yaml is now generated in your working directory.

## Create docker-compose.yaml

The next step is to create docker-compose.yaml to setup PostgreSQL, Redis, and Authgear.

You can start with the following docker-compose.yaml:

```yaml
version: "3"
services:
  db:
    image: postgres:12.3
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    ports:
      - "5432:5432"

  redis:
    image: redis:5.0
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

  authgear:
    # Remember to replace the latest tag with the exact version you would like to use!
    image: quay.io/theauthgear/authgear-server:latest
    volumes:
      - ./authgear.yaml:/app/authgear.yaml
      - ./authgear.secrets.yaml:/app/authgear.secrets.yaml
    environment:
      DEV_MODE: "true"
      LOG_LEVEL: "debug"
    ports:
      - "3000:3000"

volumes:
  redis_data:
    driver: local
  db_data:
    driver: local
```

## Edit authgear.secrets.yaml

The three services run in the same network. We have to ensure Authgear can connect to PostgreSQL and Redis.

Check authgear.secrets.yaml and see if it looks like the following:

```yaml
secrets:
- key: db
  data:
    database_schema: public
    database_url: postgres://postgres:postgres@db:5432/postgres?sslmode=disable
- key: redis
  data:
    host: redis
    port: 6379
# Other entries that are randomly generated.
# They are not listed here because they will be different.
```

## Start PostgreSQL and Redis

Authgear depends on them so they have to be started first.

```bash
docker-compose up -d db redis
```

## Run database migration

Run the database migration:

```bash
docker-compose run --rm  authgear authgear migrate up
```

## Get it running

Run everything with:

```bash
docker-compose up
```

## Verify everything is working

Visit [http://localhost:3000](http://localhost:3000) and try signing up as a new user!

