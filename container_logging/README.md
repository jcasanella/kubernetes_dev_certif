# Container Logging

https://kubernetes.io/docs/concepts/cluster-administration/logging/

## 1. We have deployed a POD hosting an application. Check the logs of the pod

```
kubectl get pods
kubectl logs webapp-1
```

## 2. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue. This is in a pod with multiple containers. The problem is with the container simple-web-app

* name of the pod: webapp-2
* container: simple-webapp

```
kubectl logs webapp-2 simple-webapp
```