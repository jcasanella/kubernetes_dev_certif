# Debugging

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/
https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/
https://github.com/kubernetes-sigs/metrics-server

## 1. Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

```
kubectl run busybox --image=busybox -- /bin/sh -c 'ls /notexist'
kubectl logs busybox
kubectl describe pod busybox
kubectl delete pod busybox
```

## 2. Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

```
kubectl run busybox --image=busybox -- /bin/sh -c 'notexist'
kubectl logs busybox
kubectl describe pod busybox
kubectl delete pods busybox --grace-period=0 --force
kubectl get pods
```

## 3. Get CPU/memory utilization for nodes (metrics-server must be running)

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
kubectl get pods -n kube-system
kubectl top nodes
kubectl top pod --namespace=kube-system
```

