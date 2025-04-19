---
title: "Basic kubectl Commands"
weight: 1
description: "An introduction to essential kubectl commands for interacting with a Kubernetes cluster."
---

`kubectl` is the command-line tool for interacting with the Kubernetes API server. It allows you to manage cluster resources, deploy applications, inspect logs, and perform many other administrative tasks.

## Common `kubectl` Operations

Here are some fundamental commands grouped by function:

### Viewing Resources

```bash
# Get a list of pods in the current namespace
kubectl get pods

# Get pods in all namespaces
kubectl get pods --all-namespaces
# Alias: kubectl get pods -A

# Get deployments in the current namespace
kubectl get deployments
# Alias: kubectl get deploy

# Get services in the current namespace
kubectl get services
# Alias: kubectl get svc

# Get nodes in the cluster
kubectl get nodes

# Get a specific pod by name
kubectl get pod my-pod-name

# Get more detailed information (e.g., Node IP, Pod IP)
kubectl get pods -o wide

# Get detailed information about a specific resource (events, status, config)
kubectl describe pod my-pod-name
kubectl describe node my-node-name
kubectl describe service my-service

# List all resource types supported by the cluster
kubectl api-resources
```

### Creating and Deleting Resources (Imperative)

While declarative management using `kubectl apply -f` is preferred for configuration management, imperative commands are useful for quick tasks and learning.

```bash
# Create a deployment running an nginx image
kubectl create deployment nginx-deployment --image=nginx

# Run a single pod (useful for testing/debugging)
# Note: kubectl run primarily creates pods now, not deployments as in older versions.
kubectl run temp-nginx --image=nginx --restart=Never # Creates a single pod

# Delete a resource by type and name
kubectl delete pod my-pod-name
kubectl delete deployment nginx-deployment
kubectl delete service my-service

# Delete resources based on a label
kubectl delete pods -l app=my-app
```

### Applying Configuration (Declarative)

This is the standard way to manage Kubernetes resources using YAML or JSON manifest files.

```bash
# Apply configuration from a file (creates or updates resources)
kubectl apply -f my-deployment.yaml
kubectl apply -f my-service.yaml

# Apply configuration from a directory
kubectl apply -f ./my-app-config/

# Delete resources defined in a file
kubectl delete -f my-deployment.yaml
```

### Debugging and Interaction

```bash
# View logs for a pod
kubectl logs my-pod-name

# Follow logs in real-time
kubectl logs -f my-pod-name

# View logs for a specific container in a multi-container pod
kubectl logs my-pod-name -c my-container-name

# Execute a command inside a container
kubectl exec my-pod-name -- ls /app # Run 'ls /app'

# Get an interactive shell inside a container
kubectl exec -it my-pod-name -- /bin/bash
```

### Configuration and Context

```bash
# View current kubectl configuration
kubectl config view

# List available contexts (clusters/users/namespaces)
kubectl config get-contexts

# Show the current context
kubectl config current-context

# Switch to a different context
kubectl config use-context my-other-context

# Set the default namespace for the current context
kubectl config set-context --current --namespace=my-namespace
```

## Alias

It's common practice to create a shell alias for `kubectl`:

```bash
alias k=kubectl
# Now you can use 'k get pods', 'k apply -f ...', etc.
```

## See Also

- {{< card title="Mastering Imperative Commands" link="4-mastering-imperative-commands-in-kubernetes" description="More advanced imperative command usage." >}}
- {{< card title="Kubernetes Namespaces" link="3-kubernetes-namespaces" description="Understanding resource scoping." >}}
- {{< card title="ReplicaSets and Deployments" link="2-understanding-replicasets-and-deployments" description="Managing application deployments." >}}

## Tags

- #Kubernetes
- #CKA
- #kubectl
- #Commands
- #CLI

## References

1. Kubernetes Documentation: [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
2. Kubernetes Documentation: [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) 
