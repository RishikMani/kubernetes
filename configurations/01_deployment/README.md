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

The `metadata` here defines the unique name of our deployment within a namespace. `namespace` separates it from resources in other namespaces. It also causes `Pods` to have naming like `web-7d8f6c9b5f-xk2lm`.

```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
        app: web
```

The deployment will have 3 replicas running at any given time. `selector` means that select resources whose label `app` has the value `web`. It identifies which `Pods` belong to it. The matching label must appear in the Pod template:

```yaml
template:
  metadata:
    labels:
      app: web
```

The template serves as the blueprint for `Pods`. Now, inside `template` there is another `spec`:

```yaml
template:
  ...
  spec:
    containers:
      - name: nginx
        image: nginx:latest
        ports:
          - containerPort: 80
```

It essentially defines a container inside each `Pod` created by the `Deployment`. The name of the container is `nginx` and it starts from image `nginx:latest`. The port `80` is opened for connections. This port, however, is not published outside the `Pod` and to reach to it a `Service` must be configured.

Now with everything done let's go ahead and deploy the `Deployment`.

```bash
kubectl apply -f configurations/01_deployment/deployment.yaml
```

List all the pods:

```bash
kubectl get pods -w
```

When the `Pods` are being created the status would should as `ContainerCreating`. The `-w` activates a watch and once the container status changes to `Running`, exit the command. All the pods will have a names starting with `web-` with random IDs assigned.

With the `Pods` in place it is time to create a `Service` to interact with the `Pods`.

## Scaling the application

When we created the `Deployment` it created three `Pods` as mentioned in

```yaml
spec:
  replicas: 3
```

Now if we forcefully delete a `Pod` to simulate real life crash, one more replica gets started to maintain the `Pod` count as 3.

```bash
kubectl delete pod web-6c79984869-8cc5k
kubectl get pods
```

Now if there are other Kubernetes components that are linked to `Pods`, like a `Service`, that too updates to reflect the changes.

## Add ConfigMap details

Once the file `configurations/03_configmap/configmap.yaml` is created and applied, we need to add the details to our `Deployment` else the changes will not be reflected. In `deployment.yaml` we modified `spec.template.spec.containers` and `spec.template.spec` sections and added volume related sections as follows:

```yaml
spec:
  ...
  template:
    ...
    spec:
      containers:
        ...
        volumeMounts:
          - name: config-content
            mountPath: /usr/share/nginx/html/index.html
            subPath: index.html
      volumes:
        - name: config-content
          configMap:
            name: web-content
```

Inside `volumeMounts` we attach the volume to the container. We then mount only the index file (`subPath`) rather than the entire volume. Finally we define the `volumes` to explicitly tell where the data comes from. This `volume` the selects the `ConfigMap` named `web-content`. If you remember, `web-content` was the name of our `ConfigMap` at the time of creation.

With this we are again ready to test the default response from NGINX. To temporarily interact with the service we could port forward as it creates a tunnel from a port on our localhost to the port in the cluster.

```bash
kubectl port-forward svc/web 8080:80
curl http://localhost:8080
```

The output HTML body should now say "Hello from Kubernetes".

Let's now jump to `Secrets`.
