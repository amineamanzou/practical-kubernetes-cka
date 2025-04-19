
up:: 
Tags:: #🌱 #Kubernetes #CKA #RBAC #Security

___

RBAC is a method for regulating access to computer or network resources based on the roles of individual users within an organization. In Kubernetes, RBAC policies are used to determine whether a request to the Kubernetes API should be allowed or denied.

**Retrieving Authorization Mode:**
Understanding the authorization mode is crucial for configuring RBAC correctly.
```shell
k describe pod -n kube-system kube-apiserver | grep authorization-mode
```

**Describing Roles and RoleBindings:**
Roles and RoleBindings are central to managing access in Kubernetes.
```shell
k describe roles
k describe rolebindings
```

**Checking User Permissions:**
You can verify what actions are allowed for a specific user.
```shell
k auth can-i get pods --as dev-user
```

**Creating Roles and RoleBindings Imperatively:**
Kubernetes allows the creation of roles and rolebindings through imperative commands.
```shell
k create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
```

**Patching Roles:**
Roles can be modified using the `patch` command.
```shell
k patch role developer -n namespace --patch-file file.yaml
```

**RBAC YAML Example:**
Here's an example of defining a Role and RoleBinding in YAML to grant specific permissions.
```yaml
# Role definition
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "create", "delete"]

# RoleBinding definition
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
```

**Key Takeaways from Lab:**
- RBAC is essential for securing your Kubernetes cluster by ensuring that users and services have only the permissions they need.
- Familiarity with `kubectl` commands related to RBAC and understanding how to define roles and rolebindings are crucial skills.

Related: 
___

Resources
1. Kubernetes Official Documentation: [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
