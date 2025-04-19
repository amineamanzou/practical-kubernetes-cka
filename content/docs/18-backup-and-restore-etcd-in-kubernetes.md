---
title: "Backup and Restore etcd in Kubernetes"
weight: 18
date: 2024-07-29 # Placeholder date
description: "Learn how to back up and restore the etcd key-value store, which holds the state of your Kubernetes cluster."
---

etcd is the distributed key-value store that serves as the backbone for Kubernetes, storing all cluster state, including configuration, specifications, and status of resources. Regularly backing up etcd is critical for disaster recovery, allowing you to restore the cluster to a known good state.

## Understanding etcd Setup

Before backup/restore, identify how etcd is running:

-   **Static Pod**: Common in `kubeadm` setups. Manifests are usually in `/etc/kubernetes/manifests/etcd.yaml` on control plane nodes.
-   **External etcd Cluster**: etcd runs on separate machines outside the Kubernetes control plane.
-   **Systemd Service**: Less common now, but etcd might run as a systemd service.

You also need connection details, typically found in the etcd or kube-apiserver static pod manifests:

-   **Endpoint(s)**: Where etcd is listening (e.g., `https://127.0.0.1:2379`).
-   **CA Certificate**: `/etc/kubernetes/pki/etcd/ca.crt`
-   **Client Certificate**: `/etc/kubernetes/pki/apiserver-etcd-client.crt` (used by API server) or specific etcd peer/client certs.
-   **Client Key**: `/etc/kubernetes/pki/apiserver-etcd-client.key`

```bash
# On a control plane node (kubeadm example)
CONTROL_PLANE_NODE="controlplane"

# Find endpoint and cert paths from etcd static pod manifest
ssh $CONTROL_PLANE_NODE sudo grep -E 'listen-client-urls|--cert-file|--key-file|--trusted-ca-file' /etc/kubernetes/manifests/etcd.yaml

# Or find API server communication certs from API server manifest
ssh $CONTROL_PLANE_NODE sudo grep -E 'etcd-servers|etcd-cafile|etcd-certfile|etcd-keyfile' /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Backing Up etcd

Use the `etcdctl snapshot save` command. Ensure you run this on a machine that can reach the etcd endpoint and has access to the necessary certificates/keys (often run directly on a control plane node or an etcd node).

```bash
# Define variables (replace with actual values found above)
ETCD_ENDPOINT="https://127.0.0.1:2379"
ETCD_CACERT="/etc/kubernetes/pki/etcd/ca.crt"
ETCD_CERT="/etc/kubernetes/pki/etcd/server.crt" # Or apiserver-etcd-client.crt
ETCD_KEY="/etc/kubernetes/pki/etcd/server.key"   # Or apiserver-etcd-client.key
BACKUP_FILE="/tmp/etcd-snapshot-$(date +%Y-%m-%d_%H-%M-%S).db"

# Run the backup command using etcdctl API v3
ETCDCTL_API=3 etcdctl \
  --endpoints=$ETCD_ENDPOINT \
  --cacert=$ETCD_CACERT \
  --cert=$ETCD_CERT \
  --key=$ETCD_KEY \
  snapshot save $BACKUP_FILE

# Verify the snapshot (optional)
ETCDCTL_API=3 etcdctl snapshot status $BACKUP_FILE
```
**Securely store the `$BACKUP_FILE` off-cluster.**

## Restoring etcd

Restoring is more disruptive and involves replacing the current etcd data directory.

**Steps:**

1.  **Stop API Server & etcd**: Prevent changes during restore.
    -   If using static pods: Temporarily move the `/etc/kubernetes/manifests/etcd.yaml` and `/etc/kubernetes/manifests/kube-apiserver.yaml` files out of the manifests directory on *all* control plane nodes.
    -   If using systemd: `sudo systemctl stop kube-apiserver` and `sudo systemctl stop etcd`.
    -   Wait for the pods/services to stop.

2.  **Run Snapshot Restore**: Use `etcdctl snapshot restore`. **Crucially, restore to a *new* data directory.**
    ```bash
    # Define variables
    SNAPSHOT_FILE="/path/to/your/etcd-snapshot.db" # Use the backup file
    NEW_DATA_DIR="/var/lib/etcd-data-new"
    
    # Run the restore command
    ETCDCTL_API=3 etcdctl snapshot restore $SNAPSHOT_FILE \
      --data-dir=$NEW_DATA_DIR
    # Optional arguments if restoring a specific member: --name, --initial-cluster, --initial-cluster-token, --initial-advertise-peer-urls
    ```

3.  **Update etcd Configuration**: Modify the etcd configuration (static pod manifest or systemd service file) to use the `NEW_DATA_DIR` for its `--data-dir` argument.
    ```bash
    # Example: Edit static pod manifest
    sudo vim /etc/kubernetes/manifests/etcd.yaml 
    # Find the --data-dir argument and change its value to /var/lib/etcd-data-new
    # Also update any volume mounts pointing to the data directory
    ```

4.  **Set Permissions**: Ensure the new data directory has the correct ownership (usually `etcd:etcd`).
    ```bash
    sudo chown -R etcd:etcd $NEW_DATA_DIR
    ```

5.  **Restart etcd & API Server**:
    -   If using static pods: Move the updated `etcd.yaml` manifest back into `/etc/kubernetes/manifests/`. Wait for etcd pod to become ready. Then move `kube-apiserver.yaml` back.
    -   If using systemd: `sudo systemctl daemon-reload`, `sudo systemctl start etcd`, `sudo systemctl start kube-apiserver`.

6.  **Verify**: Check cluster status (`kubectl get nodes`, `get pods -A`). Verify restored resources.

## Considerations

-   **Consistency**: Backups should ideally be taken when the cluster is relatively quiet if possible, though `etcdctl snapshot` provides a consistent point-in-time backup.
-   **HA Clusters**: In an HA etcd cluster, you typically back up from one member and restore involves carefully replacing the data directory on *all* members, often needing to reconfigure cluster membership if node IPs/names changed.
-   **Certificates**: Backups *do not* include the TLS certificates. These must be backed up separately or regenerated.

## See Also

- {{< card title="Static Pods" link="10-understanding-static-pods-in-kubernetes" description="How etcd is often run in kubeadm clusters." >}}
- {{< card title="Reading TLS Configurations" link="19-reading-tls-configurations-of-a-kubernetes-cluster" description="Finding the necessary certificates for etcdctl." >}}
- {{< card title="Cluster Administration" description="General cluster maintenance tasks." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #etcd
- #Backup
- #Restore
- #DisasterRecovery
- #ClusterAdministration
- #etcdctl

## References

1.  Kubernetes Documentation: [Operating etcd clusters for Kubernetes - Backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
2.  Kubernetes Documentation: [Restoring an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster)
3.  etcd Documentation: [Snapshot backup and restore](https://etcd.io/docs/v3.5/op-guide/recovery/) 