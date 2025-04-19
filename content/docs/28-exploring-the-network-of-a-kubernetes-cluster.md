---
title: "Exploring the Network of a Kubernetes Cluster"
weight: 28
date: 2024-07-29 # Placeholder date
description: "Use standard Linux networking tools to inspect interfaces, routes, and connections within Kubernetes nodes and pods."
---

Understanding the underlying network configuration within Kubernetes nodes and Pods is essential for troubleshooting connectivity issues, verifying network policy enforcement, and generally comprehending how cluster communication works. Standard Linux networking utilities are invaluable for this exploration.

## Common Linux Networking Tools

These commands can be run directly on cluster nodes (via SSH) or inside Pods (using `kubectl exec`). Note that Pods may require installing these tools or using a debug container image.

### Interface Inspection

- **`ip addr`** or **`ip a`**: Show IP addresses assigned to network interfaces.
- **`ip link`**: Show network interface details (including state UP/DOWN, MAC addresses).
- **`ifconfig`**: (Older command, may not be installed) Show interface configuration.

```bash
# On a Node
ssh <node-name> ip addr

# Inside a Pod
kubectl exec <pod-name> -- ip addr

# Look for interfaces like eth0, ensX, docker0, cni0, flannel.1, caliXXX, vethXXX, lo
```

### Routing Tables

- **`ip route`**: Show the kernel routing table (how traffic is directed).
- **`ip route get <destination-ip>`**: Show the specific route taken to reach a destination.

```bash
# On a Node
ssh <node-name> ip route

# Inside a Pod
kubectl exec <pod-name> -- ip route

# Check route to a specific pod or service IP
kubectl exec <pod-name> -- ip route get 10.244.1.5
```

### ARP / Neighbor Table

- **`ip neigh`**: Show the neighbor table (ARP cache - mapping IP addresses to MAC addresses on the local network).
- **`arp -n`**: (Older command) Show ARP cache numerically.

```bash
# On a Node
ssh <node-name> ip neigh
```

### Connections and Listening Ports

- **`ss -tulnp`**: Show listening TCP (`t`) and UDP (`u`) sockets, numeric ports (`n`), associated process (`p`), without resolving hostnames (`l`). Preferred over `netstat`.
- **`netstat -plnt`**: (Older command) Similar to `ss`, shows listening TCP ports.
- **`ss -tanp`**: Show all (`a`) TCP (`t`) connections, numeric (`n`), with process (`p`).

```bash
# On a Node: Check what kube-proxy or kubelet is listening on
ssh <node-name> ss -tulnp | grep -E 'kubelet|kube-proxy'

# Inside a Pod: Check what the application is listening on
kubectl exec <pod-name> -- ss -tulnp

# Check established connections (e.g., to etcd on port 2379 on control plane)
ssh <control-plane-node> ss -tanp | grep 2379
```

### DNS Resolution

- **`cat /etc/resolv.conf`**: Show the DNS servers and search domains configured for resolution.
- **`nslookup <hostname>`**: Query DNS for a hostname.
- **`dig <hostname>`**: (More detailed) DNS lookup utility.

```bash
# Inside a Pod: Check DNS config
kubectl exec <pod-name> -- cat /etc/resolv.conf

# Inside a Pod: Resolve a Service name
kubectl exec <pod-name> -- nslookup <service-name>.<namespace>
kubectl exec <pod-name> -- nslookup kubernetes.default
```

### Connectivity Testing

- **`ping <ip-address>`**: Check basic ICMP reachability.
- **`curl <url>`** or **`wget <url>`**: Test HTTP/HTTPS connectivity.
- **`traceroute <ip-address>`**: Trace the network path to a destination (may require installation).

```bash
# Inside a Pod: Ping another Pod IP
kubectl exec <pod-name-1> -- ping <pod-ip-2>

# Inside a Pod: Curl a Service endpoint
kubectl exec <pod-name> -- curl http://<service-name>.<namespace>:<port>
```

## Key Areas to Explore

- **Nodes**: Check physical interfaces, bridge interfaces (docker0, cni0), CNI-specific interfaces (flannel.1), routing to Pod CIDRs, kubelet/kube-proxy ports.
- **Pods**: Check `eth0` interface (usually connected to a veth pair), default gateway (often the node's bridge IP), DNS configuration, connectivity to Services and other Pods.

## See Also

- {{< card title="Exploring CNI" link="29-exploring-container-network-interface-cni-in-kubernetes" description="Understanding the CNI layer." >}}
- {{< card title="Service Networking" link="32-service-networking-in-kubernetes" description="How Services route traffic." >}}
- {{< card title="Discovering DNS" link="33-discovering-dns-in-kubernetes" description="How DNS resolution works in Kubernetes." >}}
- {{< card title="Network Policies" link="26-implementing-network-policies-in-kubernetes" description="How network policies affect connectivity." >}}

## Tags

- #Kubernetes
- #CKA
- #Networking
- #Troubleshooting
- #LinuxNetworking
- #iproute2
- #netstat
- #ss

## References

1.  Kubernetes Documentation: [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
2.  Linux Manual Pages: `ip(8)`, `ss(8)`, `netstat(8)`, `arp(8)`, `nslookup(1)` 