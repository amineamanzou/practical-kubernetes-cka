---
title: "Command and Arguments in Kubernetes Containers"
weight: 13
date: 2024-07-29 # Placeholder date
description: "Override container ENTRYPOINT and CMD using the command and args fields in Kubernetes Pod specifications."
---

Kubernetes allows you to customize the command and arguments executed when a container starts, overriding the defaults specified in the container image (typically set by the `ENTRYPOINT` and `CMD` instructions in the Dockerfile).

## Docker `ENTRYPOINT` / `CMD` vs. Kubernetes `command` / `args`

There's a direct mapping:

-   Kubernetes `spec.containers[].command` maps to Docker `ENTRYPOINT`.
-   Kubernetes `spec.containers[].args` maps to Docker `CMD`.

Understanding the interaction is key:

| Dockerfile         | Dockerfile         | Kubernetes `command` | Kubernetes `args` | Command Executed                                      |
| :----------------- | :----------------- | :------------------- | :---------------- | :---------------------------------------------------- |
| Not Set            | `["arg1", "arg2"]` | Not Set              | Not Set           | *Image Default Entrypoint* `arg1 arg2` (from CMD)     |
| Not Set            | `["arg1", "arg2"]` | Not Set              | `["arg3"]`        | *Image Default Entrypoint* `arg3` (CMD overridden)    |
| `["/ep"]`          | `["arg1", "arg2"]` | Not Set              | Not Set           | `/ep arg1 arg2` (Entrypoint + CMD)                    |
| `["/ep"]`          | `["arg1", "arg2"]` | Not Set              | `["arg3"]`        | `/ep arg3` (Entrypoint + CMD overridden)            |
| `["/ep"]`          | `["arg1", "arg2"]` | `["/cmd"]`           | Not Set           | `/cmd` (Entrypoint overridden, CMD ignored)           |
| `["/ep"]`          | `["arg1", "arg2"]` | `["/cmd"]`           | `["arg3"]`        | `/cmd arg3` (Entrypoint overridden, CMD overridden) |
| Not Set            | Not Set            | `["/cmd"]`           | `["arg3"]`        | `/cmd arg3` (Image has no defaults)                 |

**Key Rules:**

1.  If you set `command` in Kubernetes, you override the Dockerfile `ENTRYPOINT`. The Dockerfile `CMD` is ignored unless you *also* set `args`.
2.  If you only set `args` in Kubernetes, you override the Dockerfile `CMD`, and these arguments are passed to the default Dockerfile `ENTRYPOINT`.
3.  `command` and `args` in Kubernetes must be specified as arrays of strings (e.g., `["executable", "param1", "param2"]`).

## Defining `command` and `args` in YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: cmd-container-1
    image: busybox
    # Overrides default entrypoint & CMD. Runs 'echo Hello world'
    command: ["echo"]
    args: ["Hello world"]
    
  - name: cmd-container-2
    image: busybox
    # Only overrides CMD. Passes 'Hello world' to default entrypoint (/bin/sh)
    # Result depends on image entrypoint, might error or do nothing.
    args: ["Hello world"]
    
  - name: cmd-container-3
    image: busybox
    # Only overrides Entrypoint. Docker CMD ignored.
    # Runs '/bin/sh -c' with no script, likely exits quickly.
    command: ["/bin/sh", "-c"]
    
  - name: cmd-container-4
    image: busybox
    # Overrides Entrypoint & CMD. Runs '/bin/sh -c 'while true; do echo Running; sleep 5; done''
    command: ["/bin/sh", "-c"]
    args: ["while true; do echo Running; sleep 5; done"]
    
  restartPolicy: Never # Often useful for command-based pods
```

## Passing Arguments Imperatively (`kubectl run`)

You can pass arguments directly using `kubectl run` after a `--` separator.

```bash
# Run a busybox pod that executes 'echo hello world'
kubectl run echo-test --image=busybox --restart=Never -- echo "hello world"

# Run a webapp-color pod passing --color=red argument to its default entrypoint
kubectl run webapp-red --image=kodekloud/webapp-color --restart=Never -- --color=red 
```
Note: The arguments after `--` are passed as `args` to the container's entrypoint (`command`).

## Updating Command/Args

Like most Pod spec fields, `command` and `args` cannot be updated on a running Pod. You need to delete and recreate the Pod, or if managed by a controller like a Deployment, update the Deployment's template and trigger a rollout.

```bash
# Replace doesn't work for changing command/args easily
# Instead, delete and apply new definition, or edit deployment

# Example: Forcing replace (disruptive!)
# kubectl replace --force -f updated-pod-definition.yaml
```

## See Also

- {{< card title="Environment Variables" link="14-using-environment-variables-in-kubernetes" description="Alternative way to pass configuration." >}}
- {{< card title="Pod Lifecycle" description="How commands/args relate to container startup." link="#" >}} <!-- Add link -->
- {{< card title="Init Containers" link="16-understanding-init-containers-in-kubernetes" description="Running commands before main containers." >}}

## Tags

- #Kubernetes
- #CKA
- #Containers
- #Command
- #Args
- #Entrypoint
- #CMD
- #PodSpec

## References

1.  Kubernetes Documentation: [Define a Command and Arguments for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
2.  Docker Documentation: [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)
3.  Docker Documentation: [CMD](https://docs.docker.com/engine/reference/builder/#cmd) 