---
title: "Handling Certificate Signing Requests (CSR) in Kubernetes"
weight: 20
date: 2024-07-29 # Placeholder date
description: "Manage the lifecycle of TLS certificates within Kubernetes using the CertificateSigningRequest API and kubectl."
---

The Kubernetes `certificates.k8s.io` API provides a mechanism for clients to request TLS certificates signed by a Certificate Authority (CA) configured within the cluster. This allows for automated or manual issuance of certificates for users, service accounts, or components like Kubelets.

## CSR Workflow

The typical process for obtaining a certificate via the CSR API is:

1.  **Generate Private Key and CSR**: The client (user, application, node) generates a private key and a corresponding Certificate Signing Request (CSR) in PEM format. Tools like `openssl` or `cfssl` are commonly used.
    ```bash
    # Example using openssl:
    # Generate a private key
    openssl genrsa -out user-jane.key 2048
    
    # Create a CSR configuration file (user-jane.csr.conf)
    # [ req ]
    # default_bits = 2048
    # prompt = no
    # default_md = sha256
    # distinguished_name = dn
    # 
    # [ dn ]
    # CN = jane
    # O = developers # Kubernetes Group
    # 
    
    # Generate the CSR
    openssl req -new -key user-jane.key -out user-jane.csr -config user-jane.csr.conf
    ```

2.  **Create `CertificateSigningRequest` Object**: The client (or an administrator on their behalf) creates a `CertificateSigningRequest` resource in Kubernetes, embedding the base64-encoded PEM CSR.
3.  **Approve/Deny CSR**: An administrator or an automated approval controller reviews the CSR and approves or denies it using `kubectl certificate approve|deny`.
4.  **Retrieve Certificate**: Once approved, the signed certificate is populated in the `status.certificate` field of the `CertificateSigningRequest` object, base64 encoded. The client retrieves this certificate.

## Defining a `CertificateSigningRequest`

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user-jane-csr
spec:
  # Base64 encoded PEM CSR generated in step 1
  # Use: cat user-jane.csr | base64 | tr -d '\n'
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VqRVFNQTRHQTFVRUF3d0hkR1Z6ZERFTE1Ba0dBMVVFQmhNQ1ZWTXdLREFZTApFU1pBc0diTlNVRXhEUW93RGdZRFZRUUREQXhEYkc1MFpYTjBMbU52YlRFTE1Ba0dBMVVFQ2hNQ1QwVXcKSEEwREFxT1RCa05IVk5kWE9WRGZmUFVNVEFmUEJqRU1NQW9HQTFVRUF4TUxaWGhoYlhCc1pTNWpiMjB3CkhnWURWUVFEREJkZE1rc3hDekFKQmdOVkJBWVRBa05PTVNZd0pBWURWUVFMREFwVGIydGxiaTVwYnk1aApZMkZ0WlRFTE1Ba0dBMVVFQXhNQ1oyVnlkbVZ5TFdOdmJuUmxiblF3SGdZRFZRUUREQkpkWVd4bFltOXkKWlhNek1Db0dBMVVFQXd3V0lERXdNREF3TURBd01EQXdNREF3TUNBd0NRWWdDUVlEVm4wQlFRSXdSREFlCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUNJQW9DQVEwc21mQWl0VDFiSlh2ekh6am1qU25mY3lYTApQVkZ3cVcrZlRjVnhzZ0p4aXhRQ0pYcVkvK3I5bHZpZ3E1Mmh0U042RzVuVlU3NzdJcDV1MGR1dzZ4RzUKdGlqMUlNOGh0bGt6a2VwVnlzTm55a0g4Q2pCM0R3aU5nVkF2ZWZvV0I5NWVqQ3l3em5kY1dYUXlBcjQ2Cm5vVytkRkU4TkpLRlF5ck9yZ2U2dk5zQnJ2K3YrS2t2cEp5Zk9hR2NlY1h1bUJxckp5RkF2K3lUa0N0QwpmVmhjVk1QdldtT0l2eEFLd0ZBRm8zTjl6UThzUE41LzJocm9yT25IcVp1d09UdmhCcnlPclB2dGZpSFEKbStLdXU5ZWJ1cDB4M2lUaUVVUTlBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFhblRyCmV6a2Nidk5VZW54d2p5bUwvR0Z3cUpzWlpXbTcvSlZtMkh4dHJwZmZzMXJ5L2t6bW1vY1l5ZkR4Wk0wNwpoN0E4eE9wK2R4eU9hSG5VWHNqQ0c5ZzVlQk50bWlQcEZsKzZsSkY2U0p4dkV1aCtqNlRkRGR1c0hLVEcKUVZ3WjB3a3l4Zyt3aDRjNWtCakYzZHFQdFk1bU1hNUV3VmdGVEQ0QzM5NVJ1L3p0aFlxYnZCM1Jwd0xCCm82Q0Z0NWxoWXFyWUpCTm9zY1l3c2p4SEZkcU1rNzZlRDRmQ3p1a3o2am5DajR3Tmd0UUl0V1BVaGlRQQpkZnlzWnI4MVd3dzN1Wk5vWXl6TmV4c1l6MFF4T2wzdHJ3c3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  # Specifies the intended use(s) of the certificate
  usages:
  - digital signature
  - key encipherment
  - client auth
  # Specifies which signer should handle this CSR
  signerName: kubernetes.io/kube-apiserver-client
  # Optional: expirationSeconds requests a specific duration (signer may ignore)
  # expirationSeconds: 86400 # 1 day
```

- **`request`**: Base64-encoded PEM format CSR.
- **`signerName`**: Specifies which Kubernetes signer should process the request. Common built-in signers:
    - `kubernetes.io/kube-apiserver-client`: For client certificates authenticated by the API server.
    - `kubernetes.io/kube-apiserver-client-kubelet`: For client certs used by API server to connect to Kubelets.
    - `kubernetes.io/kubelet-serving`: For Kubelet server certificates.
    - `kubernetes.io/legacy-unknown`: Handled by some external CAs or controllers.
- **`usages`**: Key usage extensions requested, e.g., `digital signature`, `key encipherment`, `server auth`, `client auth`.
- **`groups`**: List of groups the requesting user belongs to (often `system:authenticated`).
- **`expirationSeconds`**: Hints at the desired certificate duration.

## Managing CSRs with `kubectl`

```bash
# List all CSRs
kubectl get csr

# Describe a specific CSR to see details and status (Pending, Approved, Denied)
kubectl describe csr <csr-name>

# Approve a pending CSR
kubectl certificate approve <csr-name>

# Deny a pending CSR
kubectl certificate deny <csr-name>

# Retrieve the signed certificate (after approval)
kubectl get csr <csr-name> -o jsonpath='{.status.certificate}' | base64 --decode > signed-cert.crt

# Delete a CSR
kubectl delete csr <csr-name>
```

## Approval Process

- **Manual Approval**: Requires a cluster administrator with permissions to approve CSRs (typically bound to the `certificatesigningrequests/approval` subresource) to manually run `kubectl certificate approve`.
- **Automatic Approval**: Kubernetes controllers (like the `kube-controller-manager`) can be configured to automatically approve certain CSRs, commonly used for Kubelet client and server certificates during node bootstrapping.

## Security Considerations

- Approving CSRs grants credentials. Ensure only trusted administrators or automated systems have permissions to approve CSRs.
- Verify the details within the CSR (CN, O, SANs, usages) before approving.
- Use RBAC to control who can create, view, and approve CSRs.

## See Also

- {{< card title="Reading TLS Configurations" link="19-reading-tls-configurations-of-a-kubernetes-cluster" description="Inspecting existing certificates." >}}
- {{< card title="RBAC" link="21-role-based-access-control-rbac-in-kubernetes" description="Controlling access to the certificates API." >}}
- {{< card title="Kubernetes Security" description="Broader security concepts." link="#" >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #CSR
- #Certificates
- #TLS
- #Security
- #PKI

## References

1.  Kubernetes Documentation: [Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
2.  Kubernetes Documentation: [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) 