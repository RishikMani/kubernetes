# ConfigMap

Create the file `configurations/03_configmap/configmap.yaml`. The `ConfigMap` changes the default index page of the NGINX to our custom defined content in `index.html`. Apply the contents of `ConfigMap`.

```bash
kubectl apply -f configurations/03_configmap/configmap.yaml
```

This overwrites the default index page of NGINX container. Once `ConfigMap` has been created add details to the `Deployment`. Please refer to `Add ConfigMap details` section in `configurations/01_deployment/deployment.yaml`.

`ConfigMap` contains only normal configurations and does not store any sensitive data like tokens, passwords or other sensitive values.
