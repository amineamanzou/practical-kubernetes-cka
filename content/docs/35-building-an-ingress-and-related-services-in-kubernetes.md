---
title: "Building an Ingress and Related Services in Kubernetes"
weight: 35
date: 2024-07-30 # Placeholder date
description: "Configure Kubernetes Ingress using Nginx to route external traffic to internal services, including namespace setup and annotations."
---

Ingress provides an API object that manages external access to the services in a cluster, typically HTTP. It can provide load balancing, SSL termination, and name-based virtual hosting. This guide covers setting up an Nginx Ingress controller and configuring Ingress resources.

# Building an Ingress and Related Services

## Isolate the Ingress Controller

It's standard practice to deploy the Ingress controller within its own dedicated namespace for better resource management and isolation.

```bash
kubectl create namespace ingress-nginx
```

## Create the Ingress Resource

The Ingress resource itself defines the rules for routing external HTTP(S) traffic to internal services. It should typically reside in the same namespace as the application services it routes to.

```bash
# Example: Get services in the target namespace
kubectl get svc -n app-space

# Example: Create an Ingress routing /wear path to wear-service on port 8080
kubectl create ingress ingress-webapp \
  --class=nginx \
  --rule="/wear=wear-service:8080" \
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/ \
  --namespace app-space
```

**Annotations:** Pay close attention to annotations, such as `nginx.ingress.kubernetes.io/rewrite-target`. These are crucial for controlling the behavior of the specific Ingress controller (like Nginx) and ensuring requests are correctly passed to backend services. The `--class` flag specifies which ingress controller should handle this Ingress object, important when multiple controllers are present.

## Expose the Ingress Controller

The Ingress controller Pods need to be exposed to receive external traffic. This is typically done via a Service of type `NodePort` or `LoadBalancer`.

**Deployment:** The controller itself runs as a Deployment. Inspecting it helps verify the image, replicas, and configuration.

```yaml
# Example snippet of an Nginx Ingress controller Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  replicas: 1 # Adjust as needed
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      serviceAccountName: ingress-nginx
      containers:
        - name: controller
          image: registry.k8s.io/ingress-nginx/controller:v1.10.1 # Use a specific version
          args:
            - /nginx-ingress-controller
            # Add necessary args like --publish-service, --ingress-class
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
            - name: metrics
              containerPort: 10254
          # ... other configurations like securityContext, resources
      # ... nodeSelector, tolerations, etc.
```

**Service:** The accompanying Service makes the controller accessible.

```yaml
# Example snippet of an Nginx Ingress controller Service
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
spec:
  type: NodePort # Or LoadBalancer if cloud provider integration is set up
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
    - name: https
      port: 443
      targetPort: https
      protocol: TCP
    # - name: metrics # Optional metrics port
    #   port: 10254
    #   targetPort: metrics
    #   protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
```

## Summary

- Deploying an Ingress controller (like Nginx) and configuring Ingress resources are fundamental for exposing Kubernetes services externally.
- Isolate the controller in its own namespace.
- Create Ingress resources in the application's namespace, using rules and annotations to define routing.
- Expose the Ingress controller using a Service (`NodePort` or `LoadBalancer`).
- Understanding annotations is crucial for correct traffic routing and behavior.

## See Also

- {{< card title="Kubernetes Services" link="#" description="Exposing applications running on Pods." >}} <!-- Add link -->
- {{< card title="Network Policies" link="#" description="Controlling traffic flow between Pods." >}} <!-- Add link -->
- {{< card title="Deployments" link="#" description="Managing application replicas." >}} <!-- Add link -->

## Tags

- #Kubernetes
- #CKA
- #Ingress
- #Services
- #Networking
- #Nginx

## References

1.  Kubernetes Official Documentation: [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
2.  Nginx Ingress Controller: [Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/) 