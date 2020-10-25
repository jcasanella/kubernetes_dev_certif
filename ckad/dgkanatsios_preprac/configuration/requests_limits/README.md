# Requests and Limits

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/

## 1. Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

```
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: 512Mi
        cpu: 200m
      requests:
        memory: 256Mi
        cpu: 100m
```