---
title: "Discovering DNS in Kubernetes"
weight: 33
date: 2024-07-29 # Placeholder date
description: "Understand how Kubernetes provides internal DNS resolution for Services and Pods using CoreDNS."
---

Kubernetes provides a built-in DNS service to enable service discovery within the cluster. Pods can resolve Services by name instead of relying on potentially changing IP addresses. This is typically managed by CoreDNS (or its predecessor, kube-dns), running as a Deployment and Service in the `kube-system` namespace.

## How Kubernetes DNS Works

1.  **DNS Service**: A Deployment (usually `coredns`) runs Pods that implement the DNS server logic. A Service (usually named `kube-dns`) provides a stable ClusterIP for these DNS Pods.
2.  **Kubelet Configuration**: Each Kubelet configures the Pods it creates to use the `kube-dns` Service IP as their nameserver. This is typically done by mounting a generated `/etc/resolv.conf` file into the Pod.
3.  **DNS Records**: CoreDNS automatically creates DNS records based on Kubernetes Services and Pods within the cluster.

## Pod DNS Configuration (`/etc/resolv.conf`)

When a Pod is created, its `/etc/resolv.conf` is configured based on the Pod's `dnsPolicy` and the cluster's DNS settings.

```bash
# Check the resolv.conf inside a pod
kubectl exec <pod-name> -- cat /etc/resolv.conf

# Example output:
# nameserver 10.96.0.10       # <-- IP of the kube-dns Service
# search my-namespace.svc.cluster.local svc.cluster.local cluster.local # <-- Search domains
# options ndots:5
```

- **`nameserver`**: Points to the ClusterIP of the `kube-dns` Service.
- **`search`**: Defines the domains that will be searched when resolving short names. Allows resolving `<service-name>` within the same namespace, or `<service-name>.<other-namespace>`.
- **`options ndots:5`**: If a name has fewer than 5 dots, the search path is tried first before querying it as an absolute name.

## DNS Record Formats

The default cluster domain is usually `cluster.local`.

- **Services (`A`/`AAAA` Records)**:
    -   `<service-name>.<namespace>.svc.<cluster-domain>` resolves to the Service's ClusterIP.
    -   Example: `my-service.default.svc.cluster.local`

- **Headless Services (`A`/`AAAA` Records)**:
    -   `<service-name>.<namespace>.svc.<cluster-domain>` resolves to the IP addresses of all ready Pods backing the Service.
    -   `<pod-hostname>.<service-name>.<namespace>.svc.<cluster-domain>` might also be created depending on Service spec.

- **Pods (`A`/`AAAA` Records)**:
    -   Usually only created if `hostname` and `subdomain` are set in the Pod spec, or if the backing Service has `publishNotReadyAddresses: true`.
    -   Format: `<pod-hostname>.<subdomain>.<namespace>.svc.<cluster-domain>`
    -   Also: `<pod-ip-address>.<namespace>.pod.<cluster-domain>` (where IP dots are replaced by dashes, e.g., `10-244-1-5.default.pod.cluster.local`)

- **Service (`SRV` Records)**:
    -   `_<port-name>._<protocol>.<service-name>.<namespace>.svc.<cluster-domain>` resolves to the port number and DNS name for named ports on a Service.
    -   Example: `_http._tcp.my-service.default.svc.cluster.local`

## Testing DNS Resolution

Use `nslookup` or `dig` inside a Pod to test resolution.

```bash
# Deploy a utility pod if needed
# kubectl run dns-test --image=busybox:1.28 --restart=Never -- sleep 3600

# Test resolving a service in the same namespace
kubectl exec dns-test -- nslookup <service-name>
# Example: kubectl exec dns-test -- nslookup kubernetes

# Test resolving a service in a different namespace
kubectl exec dns-test -- nslookup <service-name>.<other-namespace>
# Example: kubectl exec dns-test -- nslookup kube-dns.kube-system

# Test resolving the fully qualified domain name (FQDN)
kubectl exec dns-test -- nslookup <service-name>.<namespace>.svc.cluster.local

# Test resolving SRV record for a named port 'web'
kubectl exec dns-test -- nslookup -type=SRV _web._tcp.<service-name>.<namespace>.svc.cluster.local

# Test resolving Pod IP (if applicable)
kubectl exec dns-test -- nslookup <pod-ip-dashed>.<namespace>.pod.cluster.local
```

## CoreDNS Configuration

CoreDNS configuration is typically stored in a ConfigMap in the `kube-system` namespace.

```bash
# Find the CoreDNS service
kubectl get svc -n kube-system kube-dns

# Find the CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# View the CoreDNS ConfigMap
kubectl describe configmap coredns -n kube-system

# Get the CoreDNS Corefile content
kubectl get configmap coredns -n kube-system -o jsonpath='{.data.Corefile}'
```
The Corefile defines how CoreDNS handles DNS queries, including upstream resolvers, plugins (like `kubernetes`, `cache`, `forward`), etc.

## Troubleshooting

- Check if CoreDNS pods are running.
- Check CoreDNS logs: `kubectl logs -n kube-system -l k8s-app=kube-dns`.
- Verify `/etc/resolv.conf` inside the affected Pod.
- Test resolution using `nslookup` from within the Pod.
- Check NetworkPolicies aren't blocking DNS traffic (usually UDP/TCP port 53 to `kube-dns` Service IP).

## See Also

- {{< card title="Services" link="32-service-networking-in-kubernetes" description="How Services provide stable endpoints." >}}
- {{< card title="CoreDNS" description="The DNS server used by Kubernetes." link="#" >}} <!-- Add link -->
- {{< card title="Exploring Cluster Network" link="28-exploring-the-network-of-a-kubernetes-cluster" description="Using tools to inspect network details." >}}
- {{< card title="Network Policies" link="26-implementing-network-policies-in-kubernetes" description="Controlling traffic flow, including DNS." >}}

## Tags

- #Kubernetes
- #CKA
- #DNS
- #CoreDNS
- #ServiceDiscovery
- #Networking

## References

1.  Kubernetes Documentation: [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
2.  Kubernetes Documentation: [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
3.  CoreDNS Website: [coredns.io](https://coredns.io/) 