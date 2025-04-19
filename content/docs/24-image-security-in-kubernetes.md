---
title: "Image Security in Kubernetes"
weight: 24
date: 2024-07-29 # Placeholder date
description: "Securely pull container images from private registries using Docker Registry Secrets and imagePullSecrets."
---

Kubernetes needs to pull container images from registries to run applications. While public registries like Docker Hub are commonly used, organizations often use private registries for proprietary or vetted images. Kubernetes provides mechanisms to securely authenticate and pull images from these private registries.

## Image Naming and Default Registry

When you specify an image name without a registry hostname (e.g., `image: nginx`), Kubernetes defaults to pulling from the `docker.io` registry (Docker Hub). For private registries or other public ones (like `quay.io`, `gcr.io`), you must include the registry hostname in the image name:

- `image: my-private-registry.example.com/my-app:v1.2`
- `image: quay.io/prometheus/node-exporter:latest`

## Pulling from Private Registries

To pull images from a private registry that requires authentication, you need to provide credentials to the Kubelet on the node where the Pod will run.
The standard way to do this is using an `imagePullSecret`.

### 1. Create a Docker Registry Secret

First, create a Kubernetes Secret of type `kubernetes.io/dockerconfigjson`. This secret stores your registry credentials.

```bash
# Replace placeholders with your actual registry details
kubectl create secret docker-registry <secret-name> \
  --docker-server=<your-registry-server> \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email>

# Example:
kubectl create secret docker-registry my-private-reg-secret \
  --docker-server=my-private-registry.example.com \
  --docker-username=app-user \
  --docker-password='s3cr3tP@ssw0rd' \
  --docker-email=app-user@example.com
```

This command creates a Secret containing a `.dockerconfigjson` key holding the base64-encoded credentials in a format similar to the local Docker client's `~/.docker/config.json`.

### 2. Reference the Secret in a Pod

Specify the name of the created secret in the `spec.imagePullSecrets` field of your Pod definition. The Kubelet will use this secret to authenticate with the private registry when pulling the image for the Pod's containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app-pod
spec:
  containers:
  - name: my-private-app
    image: my-private-registry.example.com/my-app:v1.2
  # Reference the secret containing registry credentials
  imagePullSecrets:
  - name: my-private-reg-secret
```

### 3. Reference the Secret in a Service Account (Recommended)

Instead of adding `imagePullSecrets` to every Pod, you can add it to a Service Account. Any Pod running under that Service Account will automatically use the specified secrets for image pulls.

```bash
# Get the default service account (or the one your pods use)
kubectl get sa default

# Add the secret to the service account
kubectl patch serviceaccount default -p '{ "imagePullSecrets": [ { "name": "my-private-reg-secret" } ] }'

# Verify the secret is added
kubectl get sa default -o yaml
```

Now, any Pod created in the same namespace using the `default` Service Account (unless otherwise specified) will automatically have access to `my-private-reg-secret` for pulling images.

```yaml
# This Pod uses the default SA and automatically gets the imagePullSecret
apiVersion: v1
kind: Pod
metadata:
  name: auto-private-app-pod
spec:
  # serviceAccountName: default # Implicitly uses default if not specified
  containers:
  - name: my-private-app
    image: my-private-registry.example.com/my-app:v1.3
```

## Other Image Security Considerations

While `imagePullSecrets` handle authentication, broader image security includes:

- **Vulnerability Scanning**: Regularly scanning images for known vulnerabilities (CVEs) using tools like Trivy, Clair, or registry-integrated scanners.
- **Image Signing & Verification**: Using tools like Notary or Sigstore to sign images and configure Kubernetes to verify signatures before pulling, ensuring image integrity and provenance.
- **Minimal Base Images**: Using minimal base images (like distroless or Alpine) to reduce the attack surface.
- **Least Privilege**: Running containers with non-root users and minimal capabilities (See Security Contexts).

## See Also

- {{< card title="Secrets" link="15-managing-secrets-in-kubernetes" description="General concepts of Kubernetes Secrets." >}}
- {{< card title="Service Accounts" link="23-working-with-service-accounts-in-kubernetes" description="Identities for Pods and associating imagePullSecrets." >}}
- {{< card title="Security Contexts" link="25-configuring-security-contexts-in-kubernetes-containers" description="Securing the runtime environment of containers." >}}

## Tags

- #Kubernetes
- #CKA
- #ImageSecurity
- #Secrets
- #DockerRegistry
- #PrivateRegistry
- #imagePullSecrets
- #Security

## References

1.  Kubernetes Documentation: [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
2.  Kubernetes Documentation: [Configure Service Accounts for Pods - Add imagePullSecrets to a service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) 