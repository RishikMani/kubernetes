# Persistent Volume

A `PersistentVolume (PV)` is a storage provided to the cluster. A `PersistentVolumeClaim (PVC)` is an application's request for storage.

We define two components `PersistentVolume` and a `PersistentVolumeClaim` in our volumes definition. A `PV` is cluster wide so it has no namespace, but a `PVC` on the other hand can be part of a namespace and reserves the volume for that namespace on the `PV`.

There are different `accessModes` available.

- ReadWriteOnce (RWO): read/write by Pods on one node
- ReadOnlyMany (ROX): read-only by Pods on multiple nodes
- ReadWriteMany (RWX): read/write by Pods on multiple nodes
- ReadWriteOncePod (RWOP): read/write by only one Pod

Once the volumes have been applied we need to modify our `Deployment`. This will create `volumeMounts` and `volumes` in our containers. If our `Pod` crashes now and a new `Pod` spawns, the data would be retained between the different `Pods`.

Verify that the `PVC` got created:

```bash
kubectl get pvc web-pvc
```

It shows that the `PVC` `web-pvc` has a `Volume` `web-pv` defined. I was expecting that the volume name should be `persistent-data`. But `persistent-data` is rather part of the `Deployment` and is not the name of our `Storage` which is actually defined in `volume.yaml`.

```
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
web-pvc   Bound    web-pv   1Gi        RWO            manual         <unset>                 17d
```

Also verify the `PersistentVolume` created:

```yaml
kubectl get pv web-pv
```

The output would show the `CLAIM` to be `development/web-pvc`.

Once the storage related changes have been applied we will add them to the deployment as defined follows.

```yaml
volumes:
  - name: persistent-data
    persistentVolumeClaim:
      claimName: web-pvc
```

First, we create a volume called `persistent-data`. Then, the `volumeMounts` takes this volume and mounts it at `/usr/share/nginx/html/data`. So, for the volume called `persistent-data`, use the storage requested by the `PVC` named `web-pvc`.

```yaml
volumeMounts:
  - name: persistent-data
    mountPath: /usr/share/nginx/html/data
```

Remember, that `ConfigMap` is not copied into each `Pod` but every `Pod` can independently mount and read the same `ConfigMap`. That is suitable for configurations. A `PersistentVolume` is for data that the applications create or modify and that can survive `Pod` replacement.

Currently, our `ConfigMap` modifies the `index.html` and would override the default output for `nginx`. This configuration can be applied to multiple pods. But, if we have an application where a user uploads photo, and if the pod crashes, the uploaded data would be lost. So having a `PV` would make it persistent and it will survive pod crashes.

