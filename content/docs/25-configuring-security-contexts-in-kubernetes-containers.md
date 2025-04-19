---
title: "Configuring Security Contexts in Kubernetes Containers"
weight: 25
date: 2024-07-29 # Placeholder date
description: "Define privilege and access control settings for Pods and Containers using Security Contexts."
---

A Security Context defines privilege and access control settings for a Pod or Container. It allows you to specify security parameters like user/group IDs, Linux capabilities, privilege escalation settings, and more, helping to enforce the principle of least privilege.

## Pod vs. Container Security Context

Security settings can be applied at two levels:

1.  **Pod Level (`spec.securityContext`)**: Settings defined here apply to *all* Containers within the Pod. They also control pod-wide settings like `fsGroup`.
2.  **Container Level (`spec.containers[*].securityContext`)**: Settings defined here apply only to the specific Container. These settings override any conflicting settings defined at the Pod level.

## Common Security Context Fields

Here are some frequently used fields:

- **`runAsUser`**: Specifies the User ID (UID) that container processes should run as. If unset, it defaults to the user specified in the container image.
- **`runAsGroup`**: Specifies the primary Group ID (GID) for container processes.
- **`runAsNonRoot`**: If set to `true`, the Kubelet validates that the container does not run as UID 0 (root) before starting it. If the container tries to run as root, it will fail to start.
- **`fsGroup`**: Specifies a supplemental Group ID that applies to all containers in the Pod. Any files created in volumes mounted by the Pod will be owned by this GID. The Kubelet also changes the permission and ownership of the volume to match the `fsGroup` before mounting.
- **`readOnlyRootFilesystem`**: If `true`, mounts the container's root filesystem as read-only.
- **`allowPrivilegeEscalation`**: Controls whether a process can gain more privileges than its parent process. Defaults to `true`. Setting it to `false` ensures that `setuid` binaries or file capabilities do not grant extra privileges.
- **`capabilities`**: Fine-grained control over Linux capabilities.
    -   `add`: List of capabilities to add.
    -   `drop`: List of capabilities to remove (often used to drop `ALL` and then add back only necessary ones).
- **`seLinuxOptions`**: Configures SELinux context for the container.
- **`seccompProfile`**: Configures seccomp (secure computing mode) profiles to restrict syscalls.
- **`windowsOptions`**: Contains Windows-specific security settings.

## Examples

### Pod-Level Security Context

This example sets the `runAsUser` and `fsGroup` for all containers in the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-security-context-example
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: data-volume
    emptyDir: {}
  containers:
  - name: container-1
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: data-volume
      mountPath: /data/demo
```

### Container-Level Security Context

This example sets specific settings for one container, overriding pod-level settings if they existed. It drops all capabilities and then adds back `NET_BIND_SERVICE`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-security-context-example
spec:
  containers:
  - name: secure-container
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      runAsUser: 1001
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - "ALL"
        add:
        - "NET_BIND_SERVICE" # Allow binding to ports < 1024
```

## Verifying Security Context

You can verify the effective user or capabilities inside a running container:

```bash
# Check the user ID running processes
kubectl exec <pod-name> -c <container-name> -- id
kubectl exec <pod-name> -c <container-name> -- ps aux

# Check capabilities (requires appropriate tools in the container, e.g., capsh)
# kubectl exec <pod-name> -c <container-name> -- capsh --print
```

## See Also

- {{< card title="Pod Security Standards / Admission" description="Cluster-level policies enforcing security contexts." link="#" >}} <!-- Add link -->
- {{< card title="Capabilities" description="Linux capabilities overview." link="#" >}} <!-- Add link -->
- {{< card title="RBAC" link="21-role-based-access-control-rbac-in-kubernetes" description="Controlling who can create pods with specific security contexts." >}}

## Tags

- #Kubernetes
- #CKA
- #SecurityContext
- #Security
- #ContainerSecurity
- #PodSecurity
- #Capabilities
- #LeastPrivilege

## References

1.  Kubernetes Documentation: [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
2.  Kubernetes Documentation: [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) 