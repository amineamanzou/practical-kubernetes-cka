---
title: "Cluster Roles and Cluster Role Bindings in Kubernetes"
weight: 22
date: 2024-07-29 # Placeholder date
description: "Define and grant cluster-wide permissions using ClusterRoles and ClusterRoleBindings for managing non-namespaced resources."
---

While Roles and RoleBindings are namespaced resources for granting permissions within a specific namespace, ClusterRoles and ClusterRoleBindings operate at the cluster scope. They are used to grant permissions on cluster-scoped resources or on namespaced resources across all namespaces.

## ClusterRole

A ClusterRole contains rules that represent a set of permissions, just like a Role. However, because they are cluster-scoped, they can be used for:

- Granting permissions on non-namespaced (cluster-scoped) resources (e.g., Nodes, PersistentVolumes, Namespaces, ClusterRoles themselves).
- Granting permissions on namespaced resources (e.g., Pods, Services) across *all* namespaces.
- Granting permissions on non-resource endpoints (e.g., `/healthz`).

### Defining a ClusterRole (YAML)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # name is required
  name: cluster-resource-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["nodes", "persistentvolumes", "namespaces"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list"]
```

### Creating a ClusterRole (Imperative)

```bash
# Create a ClusterRole to view nodes
kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes

# Create a ClusterRole to manage PersistentVolumes
kubectl create clusterrole pv-manager --verb=get,list,watch,create,delete --resource=persistentvolumes

# Note: Use 'kubectl api-resources' to find the correct resource names and apiGroups
# kubectl api-resources | grep persistentvolumes
```

## ClusterRoleBinding

A ClusterRoleBinding grants the permissions defined in a ClusterRole to a set of users, groups, or service accounts across the entire cluster.

### Defining a ClusterRoleBinding (YAML)

This binds the `cluster-resource-reader` ClusterRole defined above to the user `jane`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-cluster-resources-jane
subjects:
- kind: User
  name: jane # Name is case-sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-resource-reader # Must match the name of the ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

### Creating a ClusterRoleBinding (Imperative)

```bash
# Bind the built-in 'cluster-admin' role to the user 'admin-user'
kubectl create clusterrolebinding admin-user-binding \
  --clusterrole=cluster-admin \
  --user=admin-user

# Bind the 'pv-manager' ClusterRole to the service account 'storage-admin' in the 'kube-system' namespace
kubectl create clusterrolebinding pv-manager-binding \
  --clusterrole=pv-manager \
  --serviceaccount=kube-system:storage-admin

# Bind the 'view' ClusterRole to the group 'developers'
kubectl create clusterrolebinding developers-view-binding \
  --clusterrole=view \
  --group=developers
```

## Default ClusterRoles

Kubernetes ships with several default ClusterRoles, including:

- **`cluster-admin`**: Superuser access, allows performing any action on any resource.
- **`admin`**: Full access within a namespace (when used with a RoleBinding), or full access to most resources cluster-wide except modifying cluster-level resources like Nodes or Namespaces (when used with ClusterRoleBinding - use with caution).
- **`edit`**: Allows read/write access to most objects in a namespace (via RoleBinding).
- **`view`**: Allows read-only access to see most objects in a namespace (via RoleBinding) or cluster-wide (via ClusterRoleBinding).

## When to Use ClusterRole vs. Role

- Use **ClusterRole** if you need to define permissions for:
    - Cluster-scoped resources (Nodes, PVs, Namespaces, etc.).
    - Namespaced resources across all namespaces (e.g., giving a monitoring system read access to all Pods).
    - A non-resource endpoint (like `/healthz`).
- Use **Role** if you need to define permissions for resources *within* a specific namespace.

## See Also

- {{< card title="Role-Based Access Control (RBAC)" link="21-role-based-access-control-rbac-in-kubernetes" description="Overview of Kubernetes authorization." >}}
- {{< card title="Service Accounts" link="23-working-with-service-accounts-in-kubernetes" description="Identities for processes running in Pods." >}}
- {{< card title="Kubernetes Security" description="Broader security concepts." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #RBAC
- #Security
- #Authorization
- #ClusterRole
- #ClusterRoleBinding

## References

1.  Kubernetes Documentation: [Using RBAC Authorization - Role and ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)
2.  Kubernetes Documentation: [Using RBAC Authorization - RoleBinding and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) 