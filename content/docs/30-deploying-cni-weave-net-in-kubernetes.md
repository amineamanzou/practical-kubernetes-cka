---
title: "Deploying CNI Weave Net in Kubernetes"
weight: 30
date: 2024-07-29 # Placeholder date
description: "Install the Weave Net CNI plugin in a Kubernetes cluster using the provided manifests."
---

Weave Net is a CNI plugin that provides networking and network policy for Kubernetes clusters. It creates an overlay network connecting all nodes and handles Pod networking setup. Deployment typically involves applying a single manifest file provided by Weaveworks.

## Prerequisites

-   A running Kubernetes cluster.
-   Ensure required ports are open between nodes for Weave Net communication (typically TCP 6783 and UDP 6783/6784).
-   Remove any previously installed CNI plugin to avoid conflicts.

## Installation using Manifest

The recommended way to install Weave Net is by applying the manifest hosted by Weaveworks. This manifest dynamically adjusts based on your detected Kubernetes version.

```bash
# Apply the Weave Net manifest
# This command fetches the appropriate manifest for your cluster version
kubectl apply -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml"
# Note: Check the Weave Net documentation for the latest recommended manifest URL and version.
# The older dynamic URL (https://cloud.weave.works/k8s/net?...) might still work but the GitHub release URL is often preferred.
```

## Resources Created

Applying the manifest typically creates the following resources, usually within the `kube-system` namespace:

-   **`weave-net` DaemonSet**: Ensures the main Weave Net agent pod runs on every node (including control plane nodes, unless specific tolerations/affinity rules prevent it).
-   **`weave-net` ServiceAccount**: Identity for the Weave Net pods.
-   **ClusterRole** and **ClusterRoleBinding** (e.g., `weave-net`): Grants necessary permissions to the ServiceAccount (e.g., get nodes, watch pods, manage network policies).
-   **Secrets**: May be created, especially if features like encryption are configured (though default encryption might not require explicit secrets in the manifest).
-   **NetworkPolicies**: Weave Net might install default NetworkPolicies, for example, to allow necessary traffic for its own operation.
-   **(Optional) ConfigMap**: Might be used for certain configurations, although much configuration is often passed via environment variables in the DaemonSet.

## Verifying the Installation

After applying the manifest, verify that Weave Net is running correctly:

```bash
# 1. Check the DaemonSet rollout status
kubectl rollout status ds/weave-net -n kube-system
# Wait until it reports successfully rolled out to all nodes

# 2. Check that Weave Net pods are running on all nodes
kubectl get pods -n kube-system -l name=weave-net -o wide
# Ensure all pods are in the 'Running' state

# 3. Check that nodes are Ready
kubectl get nodes
# Nodes should remain in the Ready state

# 4. (Optional) Test Pod-to-Pod Connectivity
# Deploy sample pods on different nodes and test communication between them.
# kubectl run pod1 --image=busybox --restart=Never -- sleep 3600
# kubectl run pod2 --image=busybox --restart=Never --node-name=<different-node> -- sleep 3600
# POD1_IP=$(kubectl get pod pod1 -o jsonpath='{.status.podIP}')
# kubectl exec pod2 -- ping $POD1_IP
```

## Uninstallation

To remove Weave Net, delete the resources created by the manifest:

```bash
kubectl delete -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml"
# Use the same manifest URL/file that was used for installation.
```
**Caution**: Uninstalling the CNI will break Pod networking. Ensure you have a plan to install an alternative CNI immediately if needed.

## See Also

- {{< card title="Networking with Weave" link="31-networking-with-weave-in-kubernetes" description="Features and inspection of Weave Net." >}}
- {{< card title="Exploring CNI" link="29-exploring-container-network-interface-cni-in-kubernetes" description="General CNI concepts." >}}
- {{< card title="Network Policies" link="26-implementing-network-policies-in-kubernetes" description="Weave Net enforces NetworkPolicy resources." >}}

## Tags

- #Kubernetes
- #CKA
- #CNI
- #WeaveNet
- #Networking
- #Installation
- #DaemonSet

## References

1.  Weave Net Documentation: [Integrating Kubernetes via the Addon](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)
2.  Weave Net GitHub Releases: [Weave Net Releases](https://github.com/weaveworks/weave/releases) 