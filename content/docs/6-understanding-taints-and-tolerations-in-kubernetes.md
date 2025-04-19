---
title: "Understanding Taints and Tolerations in Kubernetes"
weight: 6
date: 2024-07-29 # Placeholder date
description: "Control pod scheduling using node taints and pod tolerations in Kubernetes."
---

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. Taints are applied to nodes, marking them to repel certain pods, while tolerations are applied to pods, allowing them to schedule onto nodes with matching taints.

## Node Taints

A taint consists of a key, value, and an effect. The effect determines what happens to pods that do not tolerate the taint.

- **`NoSchedule`**: Prevents new pods from being scheduled on the node unless they tolerate the taint. Existing pods are not affected.
- **`PreferNoSchedule`**: The scheduler tries to avoid placing pods that do not tolerate the taint on the node, but it's not guaranteed.
- **`NoExecute`**: Evicts existing pods from the node if they do not tolerate the taint, and prevents new pods without the toleration from being scheduled.

### Managing Taints with `kubectl`

```bash
# View taints on a specific node
kubectl describe node <node-name> | grep Taints

# Add a taint to a node (key=value:effect)
# Example: Only pods tolerating 'app=critical:NoSchedule' can schedule
kubectl taint node <node-name> app=critical:NoSchedule

# Add a taint with only a key and effect
kubectl taint node <node-name> dedicated-gpu:NoExecute

# Update an existing taint (useful for correcting typos or changing effects)
# The key and effect must match the taint to be overwritten
kubectl taint node <node-name> app=critical:NoSchedule --overwrite

# Remove a specific taint by key and effect
kubectl taint node <node-name> app=critical:NoSchedule-

# Remove all taints with a specific key (regardless of value or effect)
kubectl taint node <node-name> app-
```

## Pod Tolerations

Tolerations are defined within a Pod's specification (`spec.tolerations`).

```yaml
spec:
  tolerations:
  - key: "app"            # Tolerates the taint key 'app'
    operator: "Equal"     # Operator can be 'Equal' or 'Exists'
    value: "critical"     # Must match the taint value if operator is 'Equal'
    effect: "NoSchedule"   # Must match the taint effect

  - key: "dedicated-gpu"  # Tolerates the taint key 'dedicated-gpu'
    operator: "Exists"    # Matches any value for the key, or no value
    effect: "NoExecute"   # Matches the taint effect

  # Toleration for NoExecute effect with a specific duration
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300 # Pod stays bound for 300s after taint is added
```

### Toleration Operators

- **`Equal`**: The `key`, `value`, and `effect` must all match the taint.
- **`Exists`**: The `key` and `effect` must match the taint. The `value` field should be omitted for the `Exists` operator.

## Use Cases

- **Dedicated Nodes**: Taint nodes to reserve them for specific workloads (e.g., GPU nodes, high-memory nodes).
- **Node Maintenance**: Apply a `NoExecute` taint to drain pods gracefully before maintenance.
- **Controlling Resource Allocation**: Prevent general workloads from running on nodes reserved for system components.

## See Also

- {{< card title="Node Affinity" link="7-implementing-node-affinity-in-kubernetes" description="Attracting pods to specific nodes." >}}
- {{< card title="Kubernetes Scheduler" link="11-configuring-multiple-schedulers-in-kubernetes" description="How Kubernetes decides where to run pods." >}}

## Tags

- #Kubernetes
- #CKA
- #Scheduling
- #Taints
- #Tolerations

## References

1.  Kubernetes Official Documentation: [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) 