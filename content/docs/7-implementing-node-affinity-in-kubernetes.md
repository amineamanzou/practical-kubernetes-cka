---
title: "Implementing Node Affinity in Kubernetes"
weight: 7
date: 2024-07-29 # Placeholder date
description: "Use node affinity rules to influence pod scheduling based on node labels in Kubernetes."
---

Node affinity allows you to constrain which nodes your Pod is eligible to be scheduled on, based on labels applied to the nodes. It provides more expressive rules compared to simple `nodeSelector`.

## Types of Node Affinity

There are two types of node affinity rules:

1.  **`requiredDuringSchedulingIgnoredDuringExecution`**: The Pod *must* be scheduled onto a node matching the specified rules. If no node matches, the Pod remains unscheduled. The scheduler ignores this rule once the Pod is running (e.g., if node labels change later, the Pod is not evicted).
2.  **`preferredDuringSchedulingIgnoredDuringExecution`**: The scheduler *tries* to find a node matching the specified rules. If a matching node is found, it's preferred. If not, the scheduler still schedules the Pod onto any available node. This rule is also ignored once the Pod is running.

## Labeling Nodes

Node affinity relies on node labels. You can add labels to nodes using `kubectl`:

```bash
# Add a label 'disktype=ssd' to node 'node01'
kubectl label node node01 disktype=ssd

# Overwrite an existing label 'color=blue' on node 'node01'
kubectl label node node01 color=blue --overwrite

# View labels on a specific node
kubectl describe node node01 | grep -i labels

# View labels for all nodes
kubectl get nodes --show-labels
```

## Defining Node Affinity in Pods

Node affinity rules are defined under `spec.affinity.nodeAffinity` in the Pod manifest.

### Required Node Affinity

This example requires the Pod to be scheduled only on nodes with the label `disktype=ssd`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-required-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In # Operator can be In, NotIn, Exists, DoesNotExist, Gt, Lt
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

### Preferred Node Affinity

This example prefers scheduling the Pod on nodes with the label `zone=us-east-1a`, but allows scheduling elsewhere if no such node is available.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-preferred-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1 # Weight (1-100) indicates preference level
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
  containers:
  - name: nginx
    image: nginx
```

### Combining Required and Preferred

You can combine both types of rules in the same `nodeAffinity` block.

## Operators for `matchExpressions`

- **`In`**: Label value must be in the provided list.
- **`NotIn`**: Label value must not be in the provided list.
- **`Exists`**: Label key must exist (value is irrelevant).
- **`DoesNotExist`**: Label key must not exist.
- **`Gt`**: Label value must be greater than the provided string (parsed as integer).
- **`Lt`**: Label value must be less than the provided string (parsed as integer).

## Checking Pod Scheduling

You can see which node a pod was scheduled on using:

```bash
kubectl get pods -o wide
```

## See Also

- {{< card title="Taints and Tolerations" link="6-understanding-taints-and-tolerations-in-kubernetes" description="Repelling pods from specific nodes." >}}
- {{< card title="Pod Affinity and Anti-Affinity" description="Scheduling pods based on labels of other pods." link="#" >}} <!-- Add link when available -->
- {{< card title="Kubernetes Scheduler" link="11-configuring-multiple-schedulers-in-kubernetes" description="How Kubernetes decides where to run pods." >}}

## Tags

- #Kubernetes
- #CKA
- #Scheduling
- #NodeAffinity
- #Affinity

## References

1.  Kubernetes Official Documentation: [Assigning Pods to Nodes - Node Affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity) 