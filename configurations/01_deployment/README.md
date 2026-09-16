# Deployment

A Kubernetes `Deployment` manages a set of `Pods` for an application, usually a stateless one. You describe the desired state, and the `Deployment` controller updates the actual state to match it, including creating `ReplicaSets` and replacing `Pods` when needed.

Let's take a look at `deployment.yaml` file, and understand some of the important settings in this file.

```yaml
kind: Deployment
```

This declares that this configuration would create a `Deployment`.

```yaml
metadata:
    name: web
    namespace: development
```

The `metadata` here defines the unique name of our deployment within a namespace. `namespace` separates it from resources in other namespaces.