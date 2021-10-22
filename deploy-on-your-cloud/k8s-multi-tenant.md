---
description: Supporting multiple tenants on Kubernetes deployment
---

# Multi-tenant mode on Kubernetes

When deployed on Kubernetes, Authgear supports hosting multiple apps as tenants. In multi- tenant mode, configuration of individual apps are stored in Kubernetes as ConfigMap/Secret, which Authgear would retrieve them at runtime.

## Deployment

Deploying Authgear in multi-tenant mode is mostly same as deploying normally. Additional configuration is needed to allow Authgear to operate in multi-tenant mode.

### Authgear namespace

The app configurations are stored in ConfigMap/Secret in a namespace; create the namespace and set its name in environment varaible `KUBE_NAMESPACE` of Authgear pod.

### Configuration source

Authgear should be configured to use Kubernetes as configuration source; set environment variable `CONFIG_SOURCE_TYPE` to `kubernetes`.

A Kubernetes service account credentials should be available to the Authgear pod, for interacting with Kubernetes resources. It should have permission to get/watch the ConfigMap/Secret resources in the specified Authgear namespace.

## Kubernetes resources

In multi-tenant mode, Authgear would use resources in Kubernetes to serve apps. Authgear would watch the resources and reload automatically when changed. There are two kind of resources to configure: Host mapping and App configuration.

### Host mapping

Host mapping is a mapping from HTTP hosts to Authgear app IDs. It is stored as a ConfigMap in Authgear namespace in Kubernetes. It should be marked with label `authgear.com/host-mapping: true`.

The ConfigMap contains single entry `hosts.json`, which content is a JSON object containing the mapping.

Multiple hosts can map to the same app ID, in case of having separate hosts for main server and admin API server.

> NOTE: `TRUST_PROXY` environment variable may needed to be set to true, if there is a reverse proxy in front of the Authgear pod.

Sample resource:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-hosts
  labels:
    authgear.com/host-mapping: "true"
data:
  hosts.json: |
    {
      "localhost:3000": "app-1",
      "example.com": "app-2",
      "admin.example.com": "app-2"
    }
```

### App configuration

App configuration is logically stored in a directory, the directory should contains `authgear.yaml` and `authgear.secrets.yaml` configuration files.

In Kubernetes, the app configuration is stored in a ConfigMap and a Secret. The data key is the relative path, and the data value is the file content. These resources should be marked with label `authgear.com/config-app-id: <app ID>`.

At runtime, the ConfigMap and Secret would be combined into a virtual directory, and app configuration files would be resolved from it.

> NOTE: App ID is stored in database along with app data, so it should not be changed after first use.
>
> NOTE: App ID in host mapping should be same as the app ID in configuration.

Sample resources:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-app-2
  labels:
    authgear.com/config-app-id: "app-2"
data:
  authgear.yaml: |
    id: my-app
    http:
      public_origin: http://example.com
---
apiVersion: v1
kind: Secret
metadata:
  name: app-config-app-2
  labels:
    authgear.com/config-app-id: "app-2"
stringData:
  authgear.secrets.yaml: |
    secrets:
    - key: db
      data:
        database_schema: app
        database_url: postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable
    - key: redis
      data:
        redis_url: redis://localhost
```
