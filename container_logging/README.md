# Container Logging

https://kubernetes.io/docs/concepts/cluster-administration/logging/
https://kubernetes.io/docs/tasks/debug-application-cluster/

Some extra commands:

* Show last line from the logs and continue showing in real time logs: `kubectl logs webapp-1 --tail 1 -f`
* Show in real time logs: `kubectl logs webapp-1 -f`

## 1. We have deployed a POD hosting an application. Check the logs of the pod

```
kubectl get pods
kubectl logs webapp-1
```

## 2. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue. This is in a pod with multiple containers. The problem is with the container simple-web-app

* name of the pod: webapp-2
* container: simple-webapp

```
kubectl logs webapp-2 -c (will print the name of the existing containers)
kubectl logs webapp-2 simple-webapp
```

**Note**: We can use selectors in logs to specify a specific object and we can combine with tail. `kubectl logs -l run=pingpong --tail 1 -f` This uses a selector called run=pingpong and tail the first line and the -f tails any new line