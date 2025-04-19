---
title: "Understanding Static Pods in Kubernetes"
weight: 10
date: 2024-07-29 # Placeholder date
description: "Learn about Static Pods, managed directly by Kubelet on specific nodes, often used for control plane components."
---

Static Pods are managed directly by the Kubelet daemon on a specific node, without the API server observing them. This means they cannot be managed via `kubectl` or other API clients directly in the way standard Pods are. However, Kubelet automatically creates a *mirror Pod* on the API server for each static Pod, making them visible (though read-only) through the API.

## How Static Pods Work

1.  **Configuration**: Kubelet is configured (either via a configuration file or command-line arguments) to watch a specific directory on the host filesystem (the *manifest path*). The default path often used by tools like `kubeadm` is `/etc/kubernetes/manifests`.
2.  **Manifest Placement**: You place standard Pod definition YAML files into this manifest path on a specific node.
3.  **Kubelet Action**: Kubelet detects these files and creates the corresponding Pods directly on that node.
4.  **Mirror Pods**: For each static Pod it runs, Kubelet attempts to create a corresponding mirror Pod object on the Kubernetes API server. This mirror Pod allows the static Pod to be discovered and viewed via `kubectl get pods`, but attempting to delete the mirror Pod using `kubectl delete` will only result in Kubelet recreating it.
5.  **Deletion**: To delete a static Pod, you must remove its manifest file from the Kubelet's configured manifest path on the relevant node. Kubelet will then automatically stop the Pod and remove its mirror Pod from the API server.

## Use Cases

- **Self-Hosted Control Plane**: Static Pods are the standard mechanism for running Kubernetes control plane components (like `kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, and `etcd`) on control plane nodes. This allows the cluster to bootstrap itself.
- **Node-Level Agents**: Running critical node-level agents or daemons that must be running even if the API server is unavailable.

## Identifying Static Pods and Configuration

```bash
# 1. Find the Kubelet process on a node (control-plane or worker)
ps aux | grep kubelet

# 2. Inspect Kubelet arguments for static pod path
# Look for '--pod-manifest-path' argument or '--config' argument pointing to a kubelet config file.
# Example using systemd config drop-in (common with kubeadm):
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Look for Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"

# 3. If using a config file, inspect it for 'staticPodPath'
# Example:
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# Output might be: staticPodPath: /etc/kubernetes/manifests

# 4. List the manifests in the identified staticPodPath
ls /etc/kubernetes/manifests
# Example output: etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

# 5. Identify mirror pods via kubectl
# Mirror pods often have a node name suffix appended and annotations
kubectl get pods -n kube-system
# Look for pods named like <manifest-name>-<node-name>

# 6. Describe a suspected mirror pod and check annotations
kubectl describe pod kube-apiserver-controlplane -n kube-system
# Look for an annotation like: kubernetes.io/config.mirror=...
# Also check ownerReferences - static pods usually have an ownerReference pointing to the Node object.

# 7. Attempting to delete a mirror pod will fail (Kubelet recreates it)
# kubectl delete pod kube-apiserver-controlplane -n kube-system # (Will be recreated)
```

## Key Considerations

- **Node Specific**: Static Pods are bound to the specific node where their manifest file resides.
- **No API Control**: They are not managed by ReplicaSets, Deployments, DaemonSets, or other controllers.
- **Resilience**: Kubelet monitors static Pods and restarts them if they fail.

## See Also

- {{< card title="DaemonSets" link="9-working-with-daemonsets-in-kubernetes" description="Running pods on all or some nodes, managed by the API server." >}}
- {{< card title="Kubernetes Control Plane" description="Understanding the core components." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #StaticPods
- #Kubelet
- #ControlPlane
- #Pods

## References

1.  Kubernetes Official Documentation: [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) 