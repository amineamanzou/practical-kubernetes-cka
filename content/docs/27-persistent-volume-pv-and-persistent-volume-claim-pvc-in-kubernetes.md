---
title: "Persistent Volume (PV) and Persistent Volume Claim (PVC) in Kubernetes"
weight: 27
date: 2024-07-29 # Placeholder date
description: "Understand the abstraction for managing persistent storage in Kubernetes using PVs and PVCs."
---

Containers in Pods often need storage that persists beyond the Pod's lifecycle. Kubernetes uses the PersistentVolume (PV) and PersistentVolumeClaim (PVC) abstraction to manage durable storage, decoupling the specifics of the underlying storage infrastructure from how Pods consume it.

## The PV/PVC Abstraction

- **PersistentVolume (PV)**: Represents a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses. PVs are cluster-scoped resources, like Nodes.
    -   They capture details of the storage implementation (NFS, iSCSI, cloud provider storage like EBS/GCEPersistentDisk, etc.).
    -   They have a lifecycle independent of any individual Pod that uses the PV.
- **PersistentVolumeClaim (PVC)**: Represents a request for storage by a user (or Pod). It's similar to how a Pod consumes Node resources; a PVC consumes PV resources. PVCs are namespace-scoped.
    -   Users request specific size and access modes without needing to know the underlying storage details.
- **Binding**: Kubernetes binds a PVC to a suitable PV based on requested `storage`, `accessModes`, and optionally `storageClassName` and `selector`. Once bound, the PVC is the exclusive user of that PV (for RWO/RWOP modes).

## PersistentVolume (PV) Definition

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv-01
spec:
  # How much storage is available
  capacity:
    storage: 5Gi # 5 Gibibytes
  # Underlying volume type and configuration
  volumeMode: Filesystem # Default (can also be Block)
  # How the volume can be mounted (ReadWriteOnce, ReadOnlyMany, ReadWriteMany, ReadWriteOncePod)
  accessModes:
    - ReadWriteOnce # Can be mounted read-write by a single node
  # What happens to the volume when the PVC is deleted (Retain, Delete, Recycle(deprecated))
  persistentVolumeReclaimPolicy: Retain
  # Optional: Links to a StorageClass for dynamic provisioning or matching
  storageClassName: standard
  # Defines the actual storage backend
  csi:
    driver: ebs.csi.aws.com # Example: AWS EBS CSI driver
    volumeHandle: vol-0abcdef1234567890 # ID of the existing EBS volume
    fsType: ext4
  # Alternative example using hostPath (for testing ONLY, not production)
  # hostPath:
  #   path: "/mnt/data/my-pv-01"
```

## PersistentVolumeClaim (PVC) Definition

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-01
  namespace: my-app-ns
spec:
  # Must match the access modes supported by the desired PV
  accessModes:
    - ReadWriteOnce
  # Minimum storage requested
  resources:
    requests:
      storage: 3Gi # Request 3 Gibibytes
  # Optional: Specify a StorageClass to request dynamic provisioning 
  # or match PVs with this StorageClass.
  storageClassName: standard
  # Optional: Specify a specific PV name for static binding (discouraged)
  # volumeName: my-pv-01
```

## Using a PVC in a Pod

Pods reference a PVC by name to mount the underlying persistent storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  namespace: my-app-ns
spec:
  containers:
    - name: my-frontend
      image: nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: web-storage
  volumes:
    - name: web-storage
      # Reference the PVC by name in the same namespace
      persistentVolumeClaim:
        claimName: my-pvc-01
```

## Key Concepts

- **Access Modes**: Define how a volume can be mounted.
    - `ReadWriteOnce` (RWO): Mount read-write by a single Node.
    - `ReadOnlyMany` (ROX): Mount read-only by many Nodes.
    - `ReadWriteMany` (RWX): Mount read-write by many Nodes.
    - `ReadWriteOncePod` (RWOP): Mount read-write by a single Pod (alpha feature, requires specific CSI drivers).
    *Note: Not all storage backends support all access modes.* 
- **Reclaim Policy**: Determines what happens to the underlying volume when the PVC is deleted.
    - `Retain`: Keeps the volume (data remains). Manual cleanup required.
    - `Delete`: Deletes the underlying storage volume (e.g., deletes EBS volume). Requires StorageClass support.
    - `Recycle` (Deprecated): Performs basic scrub (`rm -rf /thevolume/*`). Not recommended.
- **StorageClass**: Enables dynamic provisioning. When a PVC specifies a StorageClass, the corresponding provisioner creates a PV automatically.
- **Static vs. Dynamic Provisioning**: PVs can be pre-provisioned by an admin (static) or created on-demand via a StorageClass (dynamic).

## Managing PVs and PVCs

```bash
# List PersistentVolumes
kubectl get pv

# Describe a PV
kubectl describe pv <pv-name>

# List PersistentVolumeClaims in a namespace
kubectl get pvc -n <namespace>

# Describe a PVC
kubectl describe pvc <pvc-name> -n <namespace>
```

## See Also

- {{< card title="StorageClasses" description="Enabling dynamic volume provisioning." link="#" >}} <!-- Add link -->
- {{< card title="Volumes" description="General concept of volumes in Pods." link="#" >}} <!-- Add link -->
- {{< card title="StatefulSets" description="Workload API for stateful applications often using PVCs." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #Storage
- #PersistentVolume
- #PV
- #PersistentVolumeClaim
- #PVC
- #StorageClass
- #Volumes

## References

1.  Kubernetes Documentation: [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
2.  Kubernetes Documentation: [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) 