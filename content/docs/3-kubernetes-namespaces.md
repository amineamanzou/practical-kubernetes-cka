---
title: "Kubernetes Namespaces"
weight: 3
date: 2024-07-29 # Placeholder date
description: "Understand how Kubernetes Namespaces provide logical separation and resource scoping within a cluster."
---

Namespaces in Kubernetes provide a mechanism for isolating groups of resources within a single cluster. They act as a virtual cluster, scoping names for resources and enabling resource quotas and access control policies per group.

## Purpose of Namespaces

- **Scope**: Names of resources need to be unique within a namespace, but not across namespaces.
- **Organization**: Separate environments (e.g., development, staging, production) or teams/projects.
- **Resource Quotas**: Limit the amount of resources (CPU, memory, storage) that can be consumed within a namespace.
- **Access Control**: Apply RBAC policies specifically to resources within a namespace.

## Working with Namespaces (`kubectl`)

```bash
# List all namespaces in the cluster
kubectl get namespaces
# Aliases: kubectl get ns

# Create a new namespace
kubectl create namespace my-namespace
# Alternatively, use YAML:
# apiVersion: v1
# kind: Namespace
# metadata:
#   name: my-namespace

# Get resources within a specific namespace
kubectl get pods --namespace=my-namespace
kubectl get deployments -n my-namespace # -n is short for --namespace

# Get resources across all namespaces
kubectl get pods --all-namespaces
kubectl get services -A # -A is short for --all-namespaces

# Set the current context to use a specific namespace by default
kubectl config set-context --current --namespace=my-namespace
# Switch back to default namespace
kubectl config set-context --current --namespace=default

# Delete a namespace (Warning: This deletes ALL resources within it!)
kubectl delete namespace my-namespace
```

## Default Namespaces

Kubernetes starts with several initial namespaces:

- **`default`**: The default namespace for objects with no other namespace assigned.
- **`kube-system`**: For objects created by the Kubernetes system (e.g., controllers, scheduler, etcd).
- **`kube-public`**: Readable by all users (including unauthenticated). Mostly reserved for cluster usage.
- **`kube-node-lease`**: Holds Lease objects for node heartbeats.

## Namespace Scope

Not all Kubernetes resources live within a namespace.

- **Namespaced Resources**: Pods, Services, Deployments, ReplicaSets, ConfigMaps, Secrets, PersistentVolumeClaims, Roles, RoleBindings, etc.
- **Cluster-Scoped Resources**: Nodes, PersistentVolumes, Namespaces, StorageClasses, ClusterRoles, ClusterRoleBindings, etc.

```bash
# Check if a resource type is namespaced
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

## Namespace DNS

Services and Pods within the same namespace can reach each other using just their name. To reach a resource in a different namespace, you use the fully qualified domain name (FQDN):

`<service-name>.<namespace-name>.svc.cluster.local`
`<pod-ip-address>.<namespace-name>.pod.cluster.local` (Note: Pod IPs are ephemeral)

For example, a pod in the `frontend` namespace can reach a service `database-svc` in the `backend` namespace via `database-svc.backend` or `database-svc.backend.svc.cluster.local`.

## See Also

- {{< card title="Resource Quotas" description="Limiting resource consumption per namespace." link="#" >}} <!-- Add link when available -->
- {{< card title="RBAC" link="21-role-based-access-control-rbac-in-kubernetes" description="Controlling access within and across namespaces." >}}
- {{< card title="Using Selectors" link="5-using-selectors-in-kubernetes" description="Selecting resources, often within a namespace." >}}
- {{< card title="Discovering DNS" link="33-discovering-dns-in-kubernetes" description="How DNS works in Kubernetes." >}}

## Tags

- #Kubernetes
- #CKA
- #Namespaces
- #Scoping
- #MultiTenancy

## References

1.  Kubernetes Official Documentation: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 