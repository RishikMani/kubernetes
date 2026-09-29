# Secret

A Kubernetes `Secret` is an object that stores a small amount of sensitive data (like a password, token, or key) that Pods can use without putting that confidential data directly in application code or container images. Kubernetes `Secrets` are similar to `ConfigMaps`, but they are specifically intended for confidential data.

Let us first create a `Secret`. If we run the command:

```bash
kubectl create secret generic app-secret --from-literal=API_KEY=demo-key-123 -n development
```

it will create a `Secret` named `API_KEY` with the given key `demo-key-123`. This `Secret` will be created using terminal and is not reproducible. So it is better to put the definition of the `Secret` in a configuration file rather.

Let us create the file `configurations/04_secret/secret.yaml`. This too defines a `Secret` named `API_KEY` and is easier to reciprocate.

One can define more than one secrets within the `Secret` configuration as in the following YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: development
type: Opaque
stringData:
  API_KEY: demo-key-123

---
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
  namespace: development
type: Opaque
stringData:
  DB_USERNAME: admin
  DB_PASSWORD: demo-password
```

The values for `DB_USERNAME` and `DB_PASSWORD` are sensitive and this way exposed. In such cases, it would be advised not to commit this file to git. One can replace the literal string by `base64` encoded string, but it could still be easily decoded. An encoded string only makes it difficult to read the string. Other way would be to rather have an encrypted string and then decrypt it. Bit out of my reach right now.

Let's apply the secret contained in `secret.yaml`. Once done we need to modify the `Deployment` such that the `containers` get modified and look like this:

```yaml
spec:
  containers:
    - name: nginx
      ...
      env:
      - name: API_KEY
        valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_KEY
      ...
```

If there are multiple secrets defined as with `DB_USERNAME` and `DB_PASSWORD` please list each one of them under the `env`:

```yaml
env:
  - name: DATABASE_USERNAME
    valueFrom:
      secretKeyRef:
        name: database-credentials
        key: DB_USER

  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: database-credentials
        key: DB_PASSWORD
```
