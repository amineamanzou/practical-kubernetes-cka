---
title: "Managing Secrets in Kubernetes"
weight: 15
date: 2024-07-29 # Placeholder date
description: "Learn how to create, manage, and use Secrets to store sensitive data in Kubernetes."
---

Kubernetes Secrets let you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. Storing confidential information in a Secret is safer and more flexible than putting it verbatim into a Pod definition or container image.

## Creating Secrets

Secrets can be created using `kubectl` or defined in YAML manifests.

### Using `kubectl create secret`

`kubectl` provides several ways to create secrets imperatively:

- **Generic Secrets**: Create from literals, files, or environment files.

    ```bash
    # Create a secret from literal key-value pairs
    kubectl create secret generic my-db-secret --from-literal=username=admin --from-literal=password='s3cr3t'

    # Create a secret from a file (the key will be the filename, value the file content)
    # Example: Create file my-password.txt containing 's3cr3t'
    kubectl create secret generic my-file-secret --from-file=my-password.txt

    # Create a secret from a file, specifying the key name
    kubectl create secret generic my-file-key-secret --from-file=user=path/to/username.txt --from-file=pass=path/to/password.txt

    # Create a secret from an environment file (.env format)
    # Example env file (db.env): 
    # DB_USER=admin
    # DB_PASS=s3cr3t
    kubectl create secret generic my-env-secret --from-env-file=db.env
    ```

- **Docker Registry Secrets**: For pulling images from private registries.

    ```bash
    kubectl create secret docker-registry my-docker-reg-secret \
      --docker-server=<your-registry-server> \
      --docker-username=<your-username> \
      --docker-password=<your-password> \
      --docker-email=<your-email>
    ```

- **TLS Secrets**: For storing TLS certificates and keys.

    ```bash
    kubectl create secret tls my-tls-secret \
      --cert=path/to/tls.crt \
      --key=path/to/tls.key
    ```

### Defining Secrets in YAML

Secrets can also be defined declaratively. The data values must be base64 encoded.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-yaml-secret
type: Opaque # Default type
data:
  # Values must be base64 encoded
  # echo -n 'admin' | base64  => YWRtaW4=
  # echo -n 's3cr3t' | base64 => czNjcjN0
  username: YWRtaW4=
  password: czNjcjN0
```

## Using Secrets in Pods

Secrets can be consumed by Pods in several ways:

1.  **As Environment Variables**: Inject specific keys or all keys from a Secret.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-env-pod
    spec:
      containers:
      - name: mycontainer
        image: redis
        env:
          # Inject a specific key
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: my-yaml-secret
                key: username
        envFrom:
          # Inject all keys from the secret as env vars
          - secretRef:
              name: my-yaml-secret
      restartPolicy: Never
    ```

2.  **As Files in a Volume**: Mount the Secret as a volume, where each key becomes a file.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-volume-pod
    spec:
      volumes:
        - name: secret-storage
          secret:
            secretName: my-yaml-secret
      containers:
      - name: mycontainer
        image: redis
        volumeMounts:
          - name: secret-storage
            mountPath: "/etc/secret-volume"
            readOnly: true
      restartPolicy: Never
    ```

3.  **By Kubelet for Image Pulls**: Using `imagePullSecrets` in the Pod spec.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: private-reg-pod
    spec:
      containers:
      - name: private-reg-container
        image: <your-private-registry>/<image>:<tag>
      imagePullSecrets:
      - name: my-docker-reg-secret
    ```

## Security Considerations

- **Base64 is Not Encryption**: Secret data is only base64 encoded, not encrypted. Anyone with API access can retrieve it.
- **RBAC**: Use Role-Based Access Control (RBAC) to restrict access to Secrets.
- **Etcd Encryption**: Enable encryption at rest for Secrets stored in etcd.
- **Least Privilege**: Grant pods access only to the specific Secrets they need.
- **Avoid Checking into Git**: Do not commit Secret manifests with sensitive data directly into version control.

## See Also

- {{< card title="ConfigMaps" description="Managing non-sensitive configuration data." link="#" >}} <!-- Add link when available -->
- {{< card title="Service Accounts" link="23-working-with-service-accounts-in-kubernetes" description="Providing identity for pods." >}}
- {{< card title="RBAC" link="21-role-based-access-control-rbac-in-kubernetes" description="Controlling access to Kubernetes resources." >}}

## Tags

- #Kubernetes
- #CKA
- #Secrets
- #Configuration
- #Security

## References

1.  Kubernetes Official Documentation: [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
2.  Kubernetes Tasks: [Configure Pods to Use Secrets](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables) (Example uses ConfigMap but applies to Secrets) 