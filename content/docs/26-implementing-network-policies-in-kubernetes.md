---
title: "Implementing Network Policies in Kubernetes"
weight: 26
date: 2024-07-29 # Placeholder date
description: "Control network traffic flow between Pods using NetworkPolicy resources to enhance cluster security."
---

NetworkPolicy resources allow you to specify how groups of Pods are allowed to communicate with each other and with other network endpoints (like Services, external IPs). They provide firewall-like capabilities at Layer 3/4 (IP address and port) within the cluster.

**Prerequisite**: NetworkPolicies are implemented by the Container Network Interface (CNI) plugin. You must be using a CNI plugin that supports NetworkPolicy enforcement (e.g., Calico, Cilium, Weave Net, Antrea). Standard `kubenet` or `docker bridge` networking does not enforce them.

## Default Behavior

- **Pods are non-isolated by default**: They accept traffic from any source.
- **Isolation upon Selection**: As soon as *any* NetworkPolicy selects a Pod (via its `podSelector`), that Pod becomes *isolated*. This means all traffic (Ingress or Egress, depending on the policy's `policyTypes`) is **denied** unless specifically allowed by a NetworkPolicy.

## NetworkPolicy Structure

A `NetworkPolicy` specification has four main sections:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
  namespace: default
spec:
  # 1. podSelector: Which pods does this policy apply to?
  podSelector:
    matchLabels:
      app: my-app
      
  # 2. policyTypes: Does this policy affect Ingress, Egress, or both?
  policyTypes:
  - Ingress
  - Egress
  
  # 3. ingress: Rules for incoming traffic (ALLOWED traffic)
  ingress:
  - from: # List of allowed sources
      - ipBlock:
          cidr: 172.17.0.0/16
          except: [172.17.1.0/24]
      - namespaceSelector:
          matchLabels:
            project: my-project
      - podSelector:
          matchLabels:
            role: frontend
    ports: # List of allowed destination ports on the target pod
      - protocol: TCP
        port: 80
        
  # 4. egress: Rules for outgoing traffic (ALLOWED traffic)
  egress:
  - to: # List of allowed destinations
      - ipBlock:
          cidr: 10.0.0.0/24
      - podSelector:
          matchLabels:
            role: database
    ports: # List of allowed destination ports
      - protocol: TCP
        port: 5432
```

### `podSelector`

- Selects the Pods within the policy's namespace that the policy applies to.
- If empty (`{}`), it selects *all* Pods in the namespace.
- If omitted entirely, the policy applies based on context (e.g., can be used for default deny/allow rules if supported by CNI).

### `policyTypes`

- An array containing `Ingress`, `Egress`, or both.
- Determines whether the `ingress` rules, `egress` rules, or both are evaluated for the selected pods.
- If omitted:
    - Defaults to `["Ingress"]` if only `ingress` rules are specified.
    - Defaults to `["Egress"]` if only `egress` rules are specified.
    - Defaults to `["Ingress", "Egress"]` if *both* `ingress` and `egress` rules are specified.
    - **Important**: If `policyTypes` is present but empty (`[]`), *no* rules are applied, effectively isolating the pod completely if no other policies apply.

### `ingress` / `egress` Rules

- Each item in the `ingress` or `egress` array is a separate rule. Traffic is allowed if it matches *any* rule in the list.
- **`from` / `to`**: Specifies allowed sources (for ingress) or destinations (for egress). Can contain:
    - `podSelector`: Selects pods (usually in the same namespace, unless combined with `namespaceSelector`).
    - `namespaceSelector`: Selects namespaces. Traffic is allowed from/to any pod within the selected namespaces.
    - `ipBlock`: Specifies CIDR ranges for allowed source/destination IPs (useful for external traffic or node IPs).
    - If `from`/`to` is empty or omitted within a rule, it allows traffic from/to *all* sources/destinations, but restricted by the `ports` specified in that rule.
- **`ports`**: Specifies allowed protocols and ports. If omitted, allows traffic on *all* ports for the matching `from`/`to` peers.
    - `protocol`: `TCP`, `UDP`, or `SCTP`.
    - `port`: A specific port number or a named port.

## Common Examples

### Default Deny All (Ingress & Egress)

Apply to all pods in the namespace and specify no rules.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: target-ns
spec:
  podSelector: {} # Select all pods
  policyTypes:
  - Ingress
  - Egress
  # No ingress or egress rules defined = Deny all
```

### Allow Ingress from specific Namespace

Allow pods with `role=backend` to receive traffic on TCP port 8080 only from pods in namespaces labeled `project=frontend`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Allow Egress to DNS Only

Allow pods with `app=my-app` to send traffic only to UDP/TCP port 53 (DNS).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-only
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Egress
  egress:
  - to: # Omitting 'to' means allow to any destination...
    ports: # ...but only on these ports
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

## See Also

- {{< card title="Exploring CNI" link="29-exploring-container-network-interface-cni-in-kubernetes" description="Understanding the CNI plugin requirement." >}}
- {{< card title="Services" link="32-service-networking-in-kubernetes" description="Services targeted by Network Policies." >}}
- {{< card title="Namespaces" link="3-kubernetes-namespaces" description="Network Policies are namespace-scoped." >}}
- {{< card title="Labels and Selectors" link="5-using-selectors-in-kubernetes" description="Used extensively in Network Policies." >}}

## Tags

- #Kubernetes
- #CKA
- #NetworkPolicy
- #Networking
- #Security
- #Firewall
- #CNI

## References

1.  Kubernetes Documentation: [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
2.  Kubernetes Documentation: [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/) 