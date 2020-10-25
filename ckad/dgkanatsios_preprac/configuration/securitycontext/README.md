# Security Context

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

## 1. Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Edit the output to add `securityContext:`

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    runAsUser: 101
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## 2. Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Edit the output.yaml to add the `securityContext:` section

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    capabilities:
      add: ["NET_ADMIN", "SYS_TIME"]
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
