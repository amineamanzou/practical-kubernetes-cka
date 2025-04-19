---
title: "Using Environment Variables in Kubernetes"
weight: 14
date: 2024-07-29 # Placeholder date
description: "Inject configuration into containers using environment variables defined directly or sourced from ConfigMaps, Secrets, or Downward API."
---

Environment variables are a common way to pass configuration settings or sensitive data into containers running within a Pod. Kubernetes provides several methods for defining and injecting these variables.

## Defining Environment Variables

Environment variables are defined within the `env` or `envFrom` fields of a container's specification (`spec.containers[]`).

### 1. Direct Key-Value Pairs (`env`)

Define variables directly in the manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-direct-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "while true; do printenv | grep MY_; sleep 10; done"]
    env:
      - name: MY_VAR_1
        value: "simple-value"
      - name: MY_VAR_2
        value: "another-value"
```

### 2. From ConfigMap Keys (`env`)

Inject specific keys from a ConfigMap as environment variables.

```yaml
# Assumes a ConfigMap named 'my-config' exists with key 'app.mode'
# kubectl create configmap my-config --from-literal=app.mode=production

apiVersion: v1
kind: Pod
metadata:
  name: env-configmapkey-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "printenv | grep APP_MODE"]
    env:
      - name: APP_MODE
        valueFrom:
          configMapKeyRef:
            name: my-config # Name of the ConfigMap
            key: app.mode  # Key within the ConfigMap
```

### 3. From Secret Keys (`env`)

Inject specific keys from a Secret. Values are automatically decoded from base64.

```yaml
# Assumes a Secret named 'my-secret' exists with key 'api.key'
# echo -n 's3cr3t-key' | base64 => czNjcjN0LWtleQ==
# kubectl create secret generic my-secret --from-literal=api.key=czNjcjN0LWtleQ==

apiVersion: v1
kind: Pod
metadata:
  name: env-secretkey-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "printenv | grep API_KEY"]
    env:
      - name: API_KEY
        valueFrom:
          secretKeyRef:
            name: my-secret # Name of the Secret
            key: api.key   # Key within the Secret
```

### 4. From Entire ConfigMap (`envFrom`)

Inject all key-value pairs from a ConfigMap as environment variables. The key from the ConfigMap becomes the environment variable name.

```yaml
# Assumes a ConfigMap named 'my-bulk-config' exists:
# kubectl create configmap my-bulk-config --from-literal=KEY1=VAL1 --from-literal=KEY2=VAL2

apiVersion: v1
kind: Pod
metadata:
  name: envfrom-configmap-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "printenv | grep KEY"]
    envFrom:
      - configMapRef:
          name: my-bulk-config # Name of the ConfigMap
      # Optional prefix for variable names:
      # - prefix: CONFIG_
      #   configMapRef:
      #     name: my-bulk-config # Results in CONFIG_KEY1, CONFIG_KEY2
```

### 5. From Entire Secret (`envFrom`)

Inject all key-value pairs from a Secret. Values are decoded.

```yaml
# Assumes a Secret named 'my-bulk-secret' exists
# kubectl create secret generic my-bulk-secret --from-literal=USER=admin --from-literal=PASS=s3cr3t

apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "printenv | grep -E 'USER|PASS'"]
    envFrom:
      - secretRef:
          name: my-bulk-secret # Name of the Secret
      # Optional prefix:
      # - prefix: SECRET_
      #   secretRef:
      #     name: my-bulk-secret # Results in SECRET_USER, SECRET_PASS
```

### 6. From Downward API (`env`)

Expose Pod or Container metadata as environment variables.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-downwardapi-pod
  labels:
    app: myapp
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sh", "-c", "printenv | grep MY_POD"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "100m"
      limits:
        memory: "64Mi"
        cpu: "200m"
    env:
      - name: MY_POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: MY_POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: MY_POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
      - name: MY_NODE_NAME
        valueFrom:
          fieldRef:
            fieldPath: spec.nodeName
      - name: MY_CPU_REQUEST
        valueFrom:
          resourceFieldRef:
            containerName: my-container # Required for resource fields
            resource: requests.cpu
      - name: MY_MEM_LIMIT
        valueFrom:
          resourceFieldRef:
            containerName: my-container
            resource: limits.memory
```

## Verifying Environment Variables

Use `kubectl exec` to run `printenv` or `env` inside the container.

```bash
kubectl exec <pod-name> -- printenv
kubectl exec <pod-name> -- env

# Check specific variables
kubectl exec <pod-name> -- printenv | grep MY_VAR
```

## See Also

- {{< card title="ConfigMaps" description="Storing non-sensitive configuration data." link="#" >}} <!-- Add link -->
- {{< card title="Secrets" link="15-managing-secrets-in-kubernetes" description="Storing sensitive data like passwords and API keys." >}}
- {{< card title="Downward API" description="Exposing Pod and Container metadata." link="#" >}} <!-- Add link -->
- {{< card title="Container Command and Arguments" link="13-command-and-arguments-in-kubernetes-containers" description="Alternative way to pass information using command-line arguments." >}}

## Tags

- #Kubernetes
- #CKA
- #EnvironmentVariables
- #Configuration
- #ConfigMaps
- #Secrets
- #DownwardAPI
- #Pods

## References

1.  Kubernetes Documentation: [Define Environment Variables for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
2.  Kubernetes Documentation: [Expose Pod Information to Containers Through Environment Variables](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/#expose-pod-information-to-the-container-through-environment-variables)
3.  Kubernetes Documentation: [Configure Pods to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

</rewritten_file> 