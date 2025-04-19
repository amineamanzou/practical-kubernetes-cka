---
title: "Managing Resource Limits in Kubernetes"
weight: 8
date: 2024-07-29 # Placeholder date
description: "Understand and configure CPU and memory requests and limits for containers to ensure efficient resource utilization and cluster stability."
---

Kubernetes allows you to specify resource requests and limits for CPU and memory for each container within a Pod. This is essential for managing cluster resources effectively, ensuring fair resource allocation, and preventing individual workloads from impacting cluster stability.

## Requests vs. Limits

- **Requests**: The minimum amount of CPU or memory guaranteed for a container. The Kubernetes scheduler uses requests to decide which node to place a Pod on, ensuring the node has enough available capacity.
    -   If a container requests resources, it's guaranteed to get them.
- **Limits**: The maximum amount of CPU or memory a container is allowed to use.
    -   **CPU**: If a container exceeds its CPU limit, it will be throttled (its CPU usage will be capped).
    -   **Memory**: If a container exceeds its memory limit, it becomes a candidate for termination (OOMKilled - Out Of Memory Killed).

Setting requests without limits means the container has guaranteed resources but can potentially consume all available resources on the node, potentially impacting other workloads or the node itself.
Setting limits without requests provides no guarantee but caps usage.

## Resource Units

- **CPU**: Measured in CPU units. `1` CPU unit corresponds to 1 physical CPU core or 1 vCPU.
    -   Fractions are allowed, often expressed in millicores or millicpu (e.g., `500m` is 0.5 CPU).
    -   `1000m` = `1` CPU.
- **Memory**: Measured in bytes.
    -   Can be specified as plain integers or with suffixes like `Ki`, `Mi`, `Gi`, `Ti` (power of 2) or `k`, `M`, `G`, `T` (power of 10).
    -   Example: `128Mi`, `1Gi`, `256M`.

## Defining Requests and Limits

Requests and limits are specified under `spec.containers[].resources`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo-pod
spec:
  containers:
  - name: demo-container
    image: nginx
    resources:
      # Minimum guaranteed resources
      requests:
        memory: "64Mi"
        cpu: "250m" # 0.25 CPU
      # Maximum allowed resources
      limits:
        memory: "128Mi"
        cpu: "500m" # 0.5 CPU
```

## Quality of Service (QoS) Classes

Kubernetes assigns a QoS class to each Pod based on its resource requests and limits. This influences scheduling priority and how pods are handled during resource pressure (e.g., OOM killing).

1.  **Guaranteed**: Pods where *every* container has *both* memory and CPU limits specified, and *both* requests and limits are *equal* for CPU and memory.
    -   Highest priority, least likely to be killed during node resource pressure.
    ```yaml
    resources:
      requests:
        memory: "100Mi"
        cpu: "500m"
      limits:
        memory: "100Mi"
        cpu: "500m"
    ```
2.  **Burstable**: Pods where at least one container has a CPU or memory request defined, but they don't meet the criteria for Guaranteed (e.g., requests < limits, or only requests are set, or only some containers have requests/limits).
    -   Medium priority.
    ```yaml
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    ```
3.  **BestEffort**: Pods where *no* container has *any* memory or CPU requests or limits defined.
    -   Lowest priority, first to be killed during node resource pressure.

## Updating Requests/Limits

Resource requests and limits generally cannot be updated on a running Pod. To change them, you typically need to update the controller managing the Pod (e.g., Deployment, StatefulSet) and trigger a rollout, or delete and recreate the Pod if it's standalone.

```bash
# For standalone pods, delete and recreate with updated manifest
# kubectl delete pod my-pod
# kubectl apply -f updated-pod.yaml

# Or use replace --force (disruptive, not recommended for controlled workloads)
# kubectl replace --force -f updated-pod.yaml

# For Deployments, update the template and apply
# (Edit deployment YAML or use kubectl set resources)
kubectl set resources deployment my-deployment --limits=cpu=500m,memory=128Mi --requests=cpu=250m,memory=64Mi
# Then verify rollout status
kubectl rollout status deployment my-deployment
```

## Namespace Quotas and LimitRanges

- **ResourceQuota**: Administrators can set constraints on the total amount of resources (CPU, memory, storage) that can be *requested* or *limited* by all Pods within a namespace.
- **LimitRange**: Administrators can define default resource requests and limits for containers created in a namespace, as well as minimum/maximum constraints per Pod or Container.

## See Also

- {{< card title="Quality of Service (QoS)" description="Understanding Pod scheduling and eviction priorities." link="#" >}} <!-- Add link -->
- {{< card title="LimitRanges" description="Setting default resource requests/limits per namespace." link="#" >}} <!-- Add link -->
- {{< card title="ResourceQuotas" description="Constraining total resource usage per namespace." link="#" >}} <!-- Add link -->
- {{< card title="Monitoring" link="12-monitoring-in-kubernetes" description="Observing actual resource usage." >}}

## Tags

- #Kubernetes
- #CKA
- #ResourceManagement
- #ResourceLimits
- #ResourceRequests
- #CPU
- #Memory
- #QoS

## References

1.  Kubernetes Documentation: [Managing Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
2.  Kubernetes Documentation: [Assign Memory Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)
3.  Kubernetes Documentation: [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)
4.  Kubernetes Documentation: [Configure Quality of Service for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) 