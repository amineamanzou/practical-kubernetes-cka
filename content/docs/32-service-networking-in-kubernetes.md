---
title: "Service Networking in Kubernetes"
weight: 32
date: 2024-07-29 # Placeholder date
description: "Explore how Kubernetes Services provide stable endpoints for Pods and how kube-proxy enables service discovery and load balancing."
---

Kubernetes Services provide a stable networking endpoint (IP address and DNS name) for a set of Pods whose underlying IP addresses might change due to restarts or scaling. Services enable reliable communication between different parts of an application and external access.

## The Role of Services

Pods are ephemeral; they can be created and destroyed, and their IP addresses change. Services solve this problem by providing:

-   **Stable IP Address**: A `ClusterIP` that remains constant for the lifetime of the Service.
-   **Stable DNS Name**: A DNS record (`<service-name>.<namespace>.svc.cluster.local`) that resolves to the `ClusterIP`.
-   **Load Balancing**: Distributes traffic across the set of healthy Pods matching the Service's selector.
-   **Decoupling**: Pods can discover and communicate with each other via stable Service names/IPs without needing to know individual Pod IPs.

## Service Types

-   **`ClusterIP`** (Default): Exposes the Service on an internal IP address within the cluster. Reachable only from within the cluster. Used for internal communication between microservices.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-internal-service
    spec:
      selector:
        app: my-app # Selects pods with label app=my-app
      ports:
        - protocol: TCP
          port: 80 # Port the service is available on
          targetPort: 8080 # Port the container listens on
      # type: ClusterIP # Default if not specified
    ```

-   **`NodePort`**: Exposes the Service on each Node's IP at a static port (the `NodePort`). Traffic arriving at `<NodeIP>:<NodePort>` is routed to the Service's `ClusterIP`, which then load balances to backend Pods. Makes the Service accessible from outside the cluster (if node IPs are reachable).
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeport-service
    spec:
      type: NodePort
      selector:
        app: my-app
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
          # nodePort: 30007 # Optional: Specify a port (within range 30000-32767), otherwise one is allocated
    ```

-   **`LoadBalancer`**: Exposes the Service externally using a cloud provider's load balancer. The cloud provider provisions a load balancer, which directs traffic to the Service's `NodePort`s on the cluster nodes. Requires cloud provider integration.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-lb-service
    spec:
      type: LoadBalancer
      selector:
        app: my-app
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
    ```

-   **`ExternalName`**: Maps the Service to the contents of the `externalName` field (e.g., `myapp.example.com`), returning a CNAME record. No proxying or port forwarding involved. Useful for providing an internal alias to an external service.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: external-db-alias
    spec:
      type: ExternalName
      externalName: prod-db.us-east-1.rds.amazonaws.com
    ```

## How Services Work: kube-proxy

`kube-proxy` is a network proxy that runs on each node in the cluster. It watches the Kubernetes API server for changes to Service and EndpointSlice (or Endpoints) objects.
Based on these objects, `kube-proxy` configures network rules on the node (using one of several modes) to intercept traffic destined for a Service's `ClusterIP:Port` or `NodeIP:NodePort` and redirect/load balance it to one of the healthy backend Pod IPs.

**kube-proxy Modes:**

-   **`iptables`** (Common Default): Uses iptables rules for routing. Mature and widely used, but can suffer performance degradation with a very large number of services.
-   **`ipvs`**: Uses IPVS (IP Virtual Server), built on the Netfilter framework. Designed for load balancing and generally offers better performance and scalability than `iptables` mode for large clusters.
-   **`userspace`** (Deprecated): Proxied traffic through a userspace process. Inefficient and no longer recommended.
-   **`kernelspace`** (Windows): Windows equivalent using the Virtual Filtering Platform (VFP).

```bash
# Find kube-proxy pods (usually a DaemonSet in kube-system)
kubectl get pods -n kube-system -l k8s-app=kube-proxy

# Check kube-proxy logs for mode information (might show up on startup)
kubectl logs -n kube-system <kube-proxy-pod-name>

# Check kube-proxy config map (if used)
# Look for 'mode: iptables' or 'mode: ipvs'
# kubectl describe configmap kube-proxy -n kube-system
```

## Cluster IP Range

The range of IP addresses used for Service `ClusterIP`s is configured via the `--service-cluster-ip-range` flag on the `kube-apiserver`.

```bash
# Check kube-apiserver static pod manifest on control plane node
ssh <control-plane-node> sudo grep service-cluster-ip-range /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Service Discovery

Pods discover Services primarily through Kubernetes DNS.

## See Also

- {{< card title="Discovering DNS" link="33-discovering-dns-in-kubernetes" description="How services get DNS names." >}}
- {{< card title="Endpoints / EndpointSlices" description="Objects tracking ready Pod IPs for Services." link="#" >}} <!-- Add link -->
- {{< card title="kube-proxy" description="The component implementing Service routing." link="#" >}} <!-- Add link -->
- {{< card title="Ingress" link="35-building-an-ingress-and-related-services-in-kubernetes" description="Managing external HTTP/S access to Services." >}}

## Tags

- #Kubernetes
- #CKA
- #Services
- #Networking
- #kube-proxy
- #ServiceDiscovery
- #ClusterIP
- #NodePort
- #LoadBalancer

## References

1.  Kubernetes Documentation: [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
2.  Kubernetes Documentation: [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
3.  Kubernetes Documentation: [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)

</rewritten_file> 