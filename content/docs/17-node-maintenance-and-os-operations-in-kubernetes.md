---
title: "Node Maintenance and OS Operations in Kubernetes"
weight: 17
date: 2024-07-29 # Placeholder date
description: "Learn how to safely perform maintenance on Kubernetes nodes using cordon, drain, and uncordon commands."
---

Performing maintenance on Kubernetes nodes (like kernel upgrades, hardware changes, or OS patching) requires careful steps to minimize disruption to running applications. Kubernetes provides tools to gracefully remove workloads from a node before shutting it down or performing maintenance.

## The Maintenance Workflow

The typical workflow involves three main commands:

1.  **`kubectl cordon <node-name>`**: Marks the node as unschedulable. The Kubernetes scheduler will no longer place new Pods onto this node. Existing Pods continue to run.
2.  **`kubectl drain <node-name>`**: Safely evicts Pods from the node. This command first cordons the node (if not already cordoned) and then systematically terminates Pods running on it, respecting Pod Disruption Budgets (PDBs) and ensuring Pods managed by controllers (Deployments, StatefulSets, ReplicaSets) are rescheduled elsewhere. DaemonSet Pods are typically ignored by default.
3.  **(Perform Maintenance)**: After draining, you can safely shut down the node, perform OS upgrades, hardware maintenance, etc.
4.  **`kubectl uncordon <node-name>`**: Marks the node as schedulable again, allowing the scheduler to place new Pods onto it once maintenance is complete.

## Understanding `kubectl drain`

The `drain` command is crucial for gracefully removing workloads.

```bash
# Cordon the node first (optional, as drain does this)
kubectl cordon node01

# Drain the node, evicting pods
# --ignore-daemonsets: Continues even if DaemonSet pods are present (they are not evicted by default).
# --delete-emptydir-data: Forces deletion of pods using emptyDir volumes (data will be lost).
# --force: Forces deletion of pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use with caution!).
# --grace-period=-1: Set custom grace period (e.g., 60 for 60 seconds). -1 uses the pod's default.
# --timeout=0s: Set a timeout for the drain operation (e.g., 5m0s). 0 means infinite.

kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

# If drain fails due to pods without controllers or PDB violations, you might need --force
# WARNING: --force will delete pods not managed by controllers without replacement!
# kubectl drain node01 --ignore-daemonsets --force
```

### Drain Behavior

- **Respects PDBs**: `drain` will wait if evicting a Pod would violate a configured Pod Disruption Budget.
- **Ignores Unmanaged Pods**: By default, `drain` refuses to delete pods not managed by a controller (ReplicaSet, Deployment, etc.) unless `--force` is used.
- **Ignores DaemonSet Pods**: By default, `drain` ignores pods managed by DaemonSets unless `--ignore-daemonsets=false` is specified (not usually recommended as DaemonSet pods are meant to run on the node).
- **Ignores Mirror Pods**: Pods backed by static pod manifests are ignored.

## Performing OS Operations

After successfully draining a node, you can SSH into it and perform necessary OS operations like:

- Applying security patches (`apt update && apt upgrade`, `yum update`).
- Upgrading the kernel.
- Rebooting the node.
- Performing hardware maintenance.

## Bringing the Node Back Online

Once maintenance is complete and the node is ready:

```bash
# Make the node schedulable again
kubectl uncordon node01
```

The Kubernetes scheduler can now place new Pods on the node, and controllers might reschedule Pods back onto it depending on affinity rules and cluster load.

## See Also

- {{< card title="Pod Disruption Budgets (PDBs)" description="Limiting disruption during voluntary operations like node drains." link="#" >}} <!-- Add link -->
- {{< card title="Taints and Tolerations" link="6-understanding-taints-and-tolerations-in-kubernetes" description="Alternative method to repel pods using taints." >}}
- {{< card title="Cluster Upgrades" description="Managing upgrades for the entire cluster." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #NodeMaintenance
- #ClusterOperations
- #Drain
- #Cordon
- #Uncordon

## References

1.  Kubernetes Documentation: [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
2.  Kubernetes Documentation: [kubectl cordon](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cordon)
3.  Kubernetes Documentation: [kubectl uncordon](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#uncordon)
4.  Kubernetes Documentation: [kubectl drain](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#drain) 