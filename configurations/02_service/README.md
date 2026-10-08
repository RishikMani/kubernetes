# Service

Inside the `configurations/02_service` directory you will see `service.yaml`. This file consists of the service definition which will let us interact with the pods that were created by the `Deployment`.

The `service.yaml` file is very simple. It simply defines a service with a name `web` inside the development namespace. This `Service` opens up connection to the `Pods` port.

`type: NodePort` exposes a Kubernetes `Service` on a port of every node in the cluster, allowing traffic from outside the cluster to reach the selected Pods.

Let's deploy the `Service`.

```bash
kubectl apply -f configurations/02_service/service.yaml
```

Inspect the service:

```bash
kubectl get service
```

You might see an output very similar to:

```
NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
web    NodePort   10.96.2.129   <none>        80:31290/TCP   6d11h
```

To temporarily interact with the service we could port forward as it creates a tunnel from a port on our localhost to the port in the cluster. For example:

```bash
kubectl port-forward service/web 8080:80
```

This binds the port 8080 on our localhost to the pod's port. Once port forwarding is up, visit http://localhost:8080 and you should see NGINX welcome page.

## How service finds correct pods?

```bash
kubectl get pods --show-labels
```

If you run the above command you get an output similar to:

```
web-6c79984869-8cc5k   1/1     Running   2 (4h6m ago)   3d6h   app=web,pod-template-hash=6c79984869
web-6c79984869-d7sfg   1/1     Running   2 (4h6m ago)   3d6h   app=web,pod-template-hash=6c79984869
web-6c79984869-ns7s8   1/1     Running   2 (4h6m ago)   3d6h   app=web,pod-template-hash=6c79984869
```

It tells you that there are 3 pods running and each one of them have an app label of `web`. Let's be more specific this time:

```bash
kubectl get pods -o wide
```

You would see all the pods in the namespace `development` along with their IPs.

```
NAME                   READY   STATUS    RESTARTS        AGE    IP            NODE                     NOMINATED NODE   READINESS GATES
web-6c79984869-8cc5k   1/1     Running   2 (4h15m ago)   3d6h   10.244.0.19   learning-control-plane   <none>           <none>
web-6c79984869-d7sfg   1/1     Running   2 (4h15m ago)   3d6h   10.244.0.17   learning-control-plane   <none>           <none>
web-6c79984869-ns7s8   1/1     Running   2 (4h15m ago)   3d6h   10.244.0.15   learning-control-plane   <none>           <none>
```

Now if I list the endpoints of the service:

```bash
kubectl get endpointslice
```

we see

```
NAME        ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
web-rkgcr   IPv4          80      10.244.0.15,10.244.0.19,10.244.0.17   6d21h
```

Observe that all the endpoints listed are the pods with app label `web`. When we created our `Service` in its configuration we had

```yaml
spec:
  selector:
    app: web
```

The `Service` uses this selector to find matching `Pods` with app label `web` (`app=web`).

## What is a `NodePort`?

Let's first check the following:

```bash
kubectl get service web -n development
```

The output:

```
NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
web    NodePort   10.96.2.129   <none>        80:31290/TCP   21d
```

Here `80` is the `Service` port and `31290` is the `NodePort`.

With a `NodePort` `Service`, Kubernetes exposes the `Service` on each Kubernetes node at the specified port:

Kubernetes node:31290 → Service → Pods

In a traditional VM or cloud-based Kubernetes cluster, the node usually has an IP address that is reachable from outside the cluster, so clients can access the NodePort through the node's IP.

In a kind cluster, however, the Kubernetes nodes are themselves Docker containers:

localhost (host) → Docker container (Kubernetes node) → NodePort → Service → Pods

The `NodePort` is therefore available inside/on the Docker network and on the node container, but Docker does not necessarily publish that port from the node container to the host's localhost.

To access the `NodePort` through localhost:31290 on the host, the port needs to be explicitly mapped/published when creating the kind cluster, for example:

host:31290 → kind node:31290 → Service → Pods

The important distinction is that the NodePort exists and is working, but the host may have no network path to the Kubernetes node's NodePort unless Docker publishes that port.
