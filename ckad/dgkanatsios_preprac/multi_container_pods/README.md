# Multi Containers Pods

https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

## 1. Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

kubectl run busybox --image=busybox --dry-run=client -o yaml > output.yaml       

Edit output.yaml to create 2 containers:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
    resources: {}
  - image: busybox
    name: busybox2
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
cat << EOF > output.yaml
```

Remeber to add EOF at the end of the file

```
kubectl apply -f output.yaml
kubectl exec -it busybox -c busybox2 -- /bin/sh  
ls
```