---
title: "Networking with Weave in Kubernetes"
weight: 31
date: 2024-07-29 # Placeholder date
description: "Explore Weave Net, a CNI plugin providing overlay networking, service discovery, and network policy features for Kubernetes."
---

Weave Net is a popular Container Network Interface (CNI) plugin for Kubernetes that provides robust and easy-to-use networking and network policy capabilities. It creates a virtual network that connects Pods across all nodes in the cluster, enabling seamless communication without needing complex network configurations.

## Core Features of Weave Net

- **Overlay Network**: Weave Net creates an encrypted mesh overlay network, allowing Pods to communicate directly across different nodes as if they were on the same L2 network segment. It manages IP address allocation (IPAM) for Pods within the cluster.
- **Simplicity**: It generally works out-of-the-box without requiring external dependencies like a key-value store (unless using Weave IPAM without Kubernetes) or complex configurations like BGP.
- **Network Policy Enforcement**: Weave Net includes a built-in network policy controller that enforces standard Kubernetes `NetworkPolicy` resources.
- **Encryption (Optional)**: Communication between Weave peers (nodes) can be encrypted using `nacl` (fast) or `sleeve` (slower, uses AES-GCM) modes for enhanced security.
- **Service Discovery (WeaveDNS)**: While Kubernetes typically uses CoreDNS/kube-dns, Weave Net includes its own DNS component (WeaveDNS) that can provide service discovery within the Weave network. It often works alongside Kube-DNS.

## How Weave Net Works

- **DaemonSet**: Weave Net is typically deployed as a DaemonSet (e.g., `weave-net` in the `kube-system` namespace), ensuring a Weave agent runs on every node.
- **Virtual Bridge**: On each node, the Weave agent usually creates a virtual bridge (often named `weave`) and connects Pod network interfaces (veth pairs) to this bridge.
- **Peer Communication**: Weave agents on different nodes discover each other (often using the Kubernetes API or multicast) and establish encrypted tunnels (VXLAN with encryption) to forward traffic between Pods on different nodes.

## Inspecting Weave Net

```bash
# Check the Weave Net DaemonSet status
kubectl get ds -n kube-system weave-net

# Check the Weave Net Pod logs on a specific node
WEAVE_POD=$(kubectl get pods -n kube-system -l name=weave-net -o jsonpath='{.items[?(@.spec.nodeName=="<node-name>")].metadata.name}')
kubectl logs -n kube-system $WEAVE_POD -c weave

# Check network interfaces on a node (look for 'weave' bridge)
ssh <node-name> ip addr show weave
ssh <node-name> ip link | grep weave

# Check routes on a node (routes via the 'weave' interface)
ssh <node-name> ip route | grep weave

# Get detailed status from within a Weave Net pod (if tools available)
kubectl exec -n kube-system $WEAVE_POD -c weave -- /home/weave/weave --local status
```

## Considerations

- **Performance**: Overlay networks with encryption can introduce some performance overhead compared to non-overlay CNIs or unencrypted overlays.
- **Complexity**: While easy to start, understanding the overlay and troubleshooting can sometimes be more complex than simpler L3 networking models.

## See Also

- {{< card title="Deploying CNI Weave Net" link="30-deploying-cni-weave-net-in-kubernetes" description="Steps to install Weave Net." >}}
- {{< card title="Exploring CNI" link="29-exploring-container-network-interface-cni-in-kubernetes" description="General concepts of CNI plugins." >}}
- {{< card title="Network Policies" link="26-implementing-network-policies-in-kubernetes" description="Securing pod communication using NetworkPolicy (enforced by Weave)." >}}
- {{< card title="Exploring Cluster Network" link="28-exploring-the-network-of-a-kubernetes-cluster" description="General network inspection techniques." >}}

## Tags

- #Kubernetes
- #CKA
- #Networking
- #CNI
- #WeaveNet
- #OverlayNetwork
- #NetworkPolicy

## References

1.  Weave Net Documentation: [Weaveworks Docs - Net](https://www.weave.works/docs/net/latest/overview/)
2.  Kubernetes Documentation: [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) 