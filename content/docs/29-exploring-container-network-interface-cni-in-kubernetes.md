---
title: "Exploring Container Network Interface (CNI) in Kubernetes"
weight: 29
date: 2024-07-29 # Placeholder date
description: "Understanding the Container Network Interface (CNI) and its role in Kubernetes networking."
---

The Container Network Interface (CNI) is a specification and set of libraries for configuring network interfaces in Linux containers. It defines how container runtimes (like Docker or containerd) interact with network plugins to set up networking for containers.

## What is CNI?

CNI focuses solely on the network connectivity of containers and removing allocated resources when containers are deleted. It defines a simple contract between the container runtime and network plugins.

- **Runtime**: Invokes the CNI plugin when a container starts.
- **Plugin**: Responsible for creating the network interface (e.g., veth pair), assigning an IP address, and setting up routes.
- **Configuration**: Plugins are configured via JSON files, typically found in `/etc/cni/net.d/`.

## CNI in Kubernetes

Kubernetes relies on CNI plugins to provide pod networking. The Kubelet on each node is responsible for invoking the configured CNI plugin when pods are created or destroyed.

### Key Components and Locations

- **CNI Configuration Files**: Define the network configuration for pods. These are typically JSON files located in `/etc/cni/net.d/` on each worker node.
The Kubelet reads the configuration file alphabetically first in this directory to determine which plugin to use.
    ```json
    // Example: 10-calico.conflist
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          // ... Calico specific configuration ...
        },
        {
          "type": "portmap",
          "capabilities": {"portMappings": true}
        }
      ]
    }
    ```

- **CNI Plugin Binaries**: The executable files for the CNI plugins. These are typically located in `/opt/cni/bin/` on each worker node.
    Common plugins include: `bridge`, `host-local` (for IPAM), `portmap`, `flannel`, `calico`, `weave-net`, `cilium`, etc.

### Finding the Active CNI Plugin

1.  **Check Kubelet Configuration**: Examine the Kubelet arguments or configuration file to see how networking is configured (often via `--network-plugin=cni`).
    ```bash
    # Find the kubelet process and inspect its arguments
    ps -aux | grep kubelet
    # Look for --network-plugin and related CNI flags like --cni-conf-dir and --cni-bin-dir
    ```
2.  **Inspect CNI Config Directory**: List the files in `/etc/cni/net.d/`. The first file alphabetically is typically the one used by Kubelet.
    ```bash
    ls -l /etc/cni/net.d/
    # Example output might show 10-flannel.conflist, 00-weave.conflist, etc.
    ```
3.  **Check DaemonSets**: Many CNI plugins (like Calico, Flannel, Weave) run as DaemonSets in the `kube-system` namespace. Checking these can reveal the active CNI provider.
    ```bash
    kubectl get ds -n kube-system
    ```

## Troubleshooting

Understanding CNI configuration is essential for diagnosing pod networking issues, such as pods not getting IP addresses or being unable to communicate.

## See Also

- {{< card title="Deploying CNI Weave Net" link="30-deploying-cni-weave-net-in-kubernetes" description="Steps to deploy Weave Net CNI." >}}
- {{< card title="Networking with Weave" link="31-networking-with-weave-in-kubernetes" description="Exploring Weave Net features." >}}
- {{< card title="Service Networking" link="32-service-networking-in-kubernetes" description="How Services provide stable endpoints." >}}
- {{< card title="Network Policies" link="26-implementing-network-policies-in-kubernetes" description="Securing pod communication." >}}

## Tags

- #Kubernetes
- #CKA
- #Networking
- #CNI

## References

1.  Kubernetes Official Documentation: [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
2.  CNI Specification Repository: [github.com/containernetworking/cni](https://github.com/containernetworking/cni) 