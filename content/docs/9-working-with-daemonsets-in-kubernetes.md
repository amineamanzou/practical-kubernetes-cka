---
title: "Working with DaemonSets in Kubernetes"
weight: 9
date: 2024-07-29 # Placeholder date
description: "Ensure a copy of a Pod runs on all or some nodes in the cluster using DaemonSets, ideal for agents and system services."
---

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

## Use Cases for DaemonSets

DaemonSets are typically used for deploying system daemons such as:

- **Log Collectors**: Running a log collection agent like Fluentd, Logstash, or Promtail on every node.
- **Monitoring Agents**: Deploying node monitoring agents like Prometheus Node Exporter, Datadog Agent, or Dynatrace OneAgent.
- **Cluster Storage**: Running cluster storage daemons like `glusterd` or `ceph` on each node.
- **Networking Plugins**: CNI network plugins like Calico, Flannel, or Weave Net often run as DaemonSets.
- **Node Problem Detectors**: Tools that monitor node health and report issues.

## How DaemonSets Work

The DaemonSet controller creates Pods on nodes based on the DaemonSet definition.

- **Node Selection**: By default, a Pod is created on every node. You can restrict this using `spec.template.spec.nodeSelector` or `spec.template.spec.affinity.nodeAffinity`.
- **Scheduling**: DaemonSet Pods bypass the default Kubernetes scheduler. The DaemonSet controller manages their placement directly. Taints and Tolerations are still respected – the controller will only schedule pods on nodes that tolerate the DaemonSet pod's tolerations if the node has matching taints.
- **Uniqueness**: Ensures only one Pod instance runs per selected node.

## Defining a DaemonSet

A DaemonSet is defined using a YAML manifest.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system # Often placed in kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      # Ensure the pod can run on control-plane nodes if needed
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      # Or use specific tolerations if targeting tainted nodes
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2 # Use appropriate image
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        # Mount host paths if needed (e.g., for log collection)
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Managing DaemonSets with `kubectl`

```bash
# List DaemonSets in the current namespace
kubectl get daemonsets
# Alias: kubectl get ds

# List DaemonSets in all namespaces
kubectl get ds -A

# Describe a DaemonSet to see its status and configuration
kubectl describe ds <daemonset-name> -n <namespace>
# Example:
kubectl describe ds kube-proxy -n kube-system

# Get the YAML definition of an existing DaemonSet
kubectl get ds <daemonset-name> -n <namespace> -o yaml

# Edit a DaemonSet (opens default editor)
kubectl edit ds <daemonset-name> -n <namespace>

# Delete a DaemonSet (also deletes its Pods)
kubectl delete ds <daemonset-name> -n <namespace>
```

## Update Strategies

DaemonSets support two update strategies, specified in `spec.updateStrategy.type`:

- **`RollingUpdate`** (Default): Gradually replaces old Pods with new ones, node by node. You can control the process with `spec.updateStrategy.rollingUpdate.maxUnavailable` (how many nodes can have unavailable pods during the update).
- **`OnDelete`**: Does not automatically update Pods when the DaemonSet template changes. New Pods are only created with the new template when old Pods are manually deleted or nodes are added.

## DaemonSet vs Deployment vs Static Pod

- **DaemonSet**: Ensures one Pod per selected node. Managed by API server. Ideal for node-level agents.
- **Deployment**: Manages a set of replicated Pods, distributing them across nodes. Managed by API server. Ideal for stateless applications.
- **Static Pod**: Runs one Pod on a specific node. Managed by Kubelet directly (not API server). Ideal for control plane components.

## See Also

- {{< card title="Static Pods" link="10-understanding-static-pods-in-kubernetes" description="Pods managed directly by Kubelet." >}}
- {{< card title="Deployments" link="2-understanding-replicasets-and-deployments" description="Managing replicated stateless applications." >}}
- {{< card title="Node Affinity" link="7-implementing-node-affinity-in-kubernetes" description="Controlling which nodes DaemonSet pods run on." >}}
- {{< card title="Taints and Tolerations" link="6-understanding-taints-and-tolerations-in-kubernetes" description="Preventing/allowing DaemonSet pods on specific nodes." >}}

## Tags

- #Kubernetes
- #CKA
- #DaemonSets
- #Workloads
- #Agents

## References

1.  Kubernetes Official Documentation: [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 