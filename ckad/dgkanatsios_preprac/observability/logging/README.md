# Logging

https://kubernetes.io/docs/tasks/debug-application-cluster/
https://kubernetes.io/docs/concepts/cluster-administration/logging/

## 1. Create a busybox pod that runs 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'. Check its logs

```
kubectl run busybox --image=busybox -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'

kubectl logs busybox -f
```
