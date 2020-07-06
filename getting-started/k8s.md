---
description: Guidelines on deployment on Kubernetes
---

# Deploying on Kubernetes

Authgear is distributed as a Docker image so it can run on Kubernetes.

## Prerequisite

First you need to prepare an instance of PostgreSQL and Redis.

### PostgreSQL

If you are an user of cloud provider, you are recommended to create an instance of cloud database, such as Amazon RDS, Google Cloud SQL, etc.

### Redis

Sessions are stored in Redis so the data of Redis should be persistent.
Some cloud providers offer managed Redis instances with persistence.
Alternatively, if you are using Helm, you can consider using some well-maintained Redis helm charts.

## authgear.yaml

It is a configuration file without any secrets so it should be a ConfigMap.

## authgear.secrets.yaml

It contains secrets and credentials so it should be a Secret.

## Scalability

Authgear stores the state in PostgreSQL and Redis so it can be easily horizontally scaled.
Refer to [the documentation of Kubernetes on scaling](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/) for more details.
