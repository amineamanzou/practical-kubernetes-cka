---
title: "Working with Service Accounts in Kubernetes"
weight: 23
date: 2024-07-29 # Placeholder date
description: "Understand how Service Accounts provide an identity for processes within Pods to interact securely with the Kubernetes API."
---

Service Accounts (SAs) provide an identity for processes running inside Pods. They are distinct from User Accounts, which are intended for humans interacting with the cluster. Applications running within Pods can use their Service Account credentials to authenticate to the Kubernetes API server, enabling them to list, create, or modify resources based on their assigned permissions (RBAC).

## Default Service Account

Every namespace has a default Service Account named `default`. If a Pod definition doesn't specify a `serviceAccountName`, it automatically uses the `default` SA in its namespace.

## Assigning Service Accounts to Pods

You can specify a non-default Service Account for a Pod using the `spec.serviceAccountName` field:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  namespace: app-ns
spec:
  # Assign the 'my-app-sa' service account to this pod
  serviceAccountName: my-app-sa 
  containers:
  - name: my-app-container
    image: my-app:latest
```

## Service Account Tokens

When a Pod uses a Service Account, Kubernetes automatically mounts a credential (a time-limited, audience-bound JWT token) into the Pod at `/var/run/secrets/kubernetes.io/serviceaccount/token`. Client libraries (like client-go) typically use this token automatically when making API calls from within the cluster.

- **Automatic Mounting**: By default, `automountServiceAccountToken` is `true` for both the Service Account and the Pod.
- **Disabling Auto-Mount**: You can disable this mounting per-Pod (`spec.automountServiceAccountToken: false`) or per-Service Account (`automountServiceAccountToken: false` on the SA object) if the Pod doesn't need API access.

## Managing Service Accounts

```bash
# List Service Accounts in the current namespace
kubectl get serviceaccounts
# Alias: kubectl get sa

# List Service Accounts in a specific namespace
kubectl get sa -n kube-system

# Create a new Service Account
kubectl create serviceaccount my-app-sa -n app-ns

# Describe a Service Account (shows associated secrets/tokens)
kubectl describe sa my-app-sa -n app-ns

# Delete a Service Account
kubectl delete sa my-app-sa -n app-ns
```

## Requesting Tokens Manually (`kubectl create token`)

Since Kubernetes 1.24, the recommended way to get a token for an SA (e.g., for use outside the cluster or for testing) is using `kubectl create token`. These tokens are time-limited by default.

```bash
# Create a token for 'my-app-sa' in namespace 'app-ns' (default expiration: 1 hour)
kubectl create token my-app-sa -n app-ns

# Request a token with a specific duration (e.g., 10 minutes)
kubectl create token my-app-sa -n app-ns --duration=10m

# Request a token bound to a specific audience (advanced)
# kubectl create token my-app-sa -n app-ns --audience=api.example.com

# Decode the token (JWT) to inspect its contents (payload)
TOKEN=$(kubectl create token my-app-sa -n app-ns)
# Requires jq
# echo $TOKEN | cut -d '.' -f 2 | base64 -d | jq .
```

## Legacy Token Generation via Secrets (Discouraged)

Before Kubernetes 1.24, SA tokens were primarily long-lived credentials stored directly in Secrets. While this mechanism still exists for backward compatibility, creating non-expiring Secret-based tokens manually is generally discouraged due to security implications.

```yaml
# Example of a manually created Secret for an SA token (Legacy/Discouraged)
# apiVersion: v1
# kind: Secret
# metadata:
#   name: my-app-sa-manual-token
#   namespace: app-ns
#   annotations:
#     kubernetes.io/service-account.name: my-app-sa
# type: kubernetes.io/service-account-token
```
Kubernetes no longer automatically creates these secrets for new Service Accounts.

## Granting Permissions (RBAC)

Creating a Service Account only provides an identity. To allow it to perform actions, you must grant it permissions using Roles/ClusterRoles and RoleBindings/ClusterRoleBindings.

```bash
# Example: Grant 'view' ClusterRole to 'my-app-sa' in 'app-ns'
kubectl create rolebinding my-app-sa-view-binding \
  --clusterrole=view \
  --serviceaccount=app-ns:my-app-sa \
  --namespace=app-ns
```

## Image Pull Secrets

Service Accounts can also be associated with `imagePullSecrets` to allow Pods using the SA to pull images from private registries without specifying the secret in each Pod definition.

```bash
# Patch the SA to include an imagePullSecret
kubectl patch serviceaccount my-app-sa -n app-ns -p '{"imagePullSecrets": [{"name": "my-private-reg-secret"}]}'
```

## See Also

- {{< card title="Role-Based Access Control (RBAC)" link="21-role-based-access-control-rbac-in-kubernetes" description="Granting permissions to Service Accounts." >}}
- {{< card title="Cluster Roles and Bindings" link="22-cluster-roles-and-cluster-role-bindings-in-kubernetes" description="Granting cluster-wide permissions." >}}
- {{< card title="Secrets" link="15-managing-secrets-in-kubernetes" description="How SA tokens are stored (legacy) and general secret management." >}}
- {{< card title="Image Security" link="24-image-security-in-kubernetes" description="Using imagePullSecrets with Service Accounts." >}}

## Tags

- #Kubernetes
- #CKA
- #ServiceAccounts
- #Security
- #Authentication
- #Authorization
- #RBAC
- #Identity

## References

1.  Kubernetes Documentation: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
2.  Kubernetes Documentation: [Managing Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)
3.  Kubernetes Documentation: [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) 