# Lighting Labs 2

## 1. We have deployed a few pods in this cluster in various namespaces. 

* Inspect them and identify the pod which is not in a **Ready state**. Troubleshoot and fix the issue.
* Next, add a check to restart the container on the same pod if the command ls /var/www/html/file_check fails. This check should start after a delay of 10 seconds and run every 60 seconds.
* You may delete and recreate the object. Ignore the warnings from the probe.

```
kubectl get pods --all-namespaces
```

```
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
default       dev-pod-dind-878516                    3/3     Running   0          2m55s
default       pod-xyz1123                            1/1     Running   0          2m56s
dev0403       nginx0403                              1/1     Running   0          2m56s
dev0403       pod-dar85                              1/1     Running   0          2m55s
dev1401       nginx1401                              0/1     Running   0          2m56s
dev1401       pod-kab87                              1/1     Running   0          2m55s
dev2406       nginx2406                              1/1     Running   0          2m56s
dev2406       pod-var2016                            1/1     Running   0          2m56s
kube-system   coredns-f9fd979d6-86csq                1/1     Running   0          3m21s
kube-system   coredns-f9fd979d6-qwtbc                1/1     Running   0          3m20s
kube-system   etcd-controlplane                      1/1     Running   0          3m30s
kube-system   kube-apiserver-controlplane            1/1     Running   0          3m30s
kube-system   kube-controller-manager-controlplane   1/1     Running   0          3m30s
kube-system   kube-flannel-ds-amd64-4hcg7            1/1     Running   0          3m21s
kube-system   kube-flannel-ds-amd64-nk5r9            1/1     Running   0          3m7s
kube-system   kube-proxy-8859n                       1/1     Running   0          3m7s
kube-system   kube-proxy-d22zg                       1/1     Running   0          3m21s
kube-system   kube-scheduler-controlplane            1/1     Running   0          3m30s
```

The pod `dev1401       nginx1401                              0/1     Running   0          2m56s` has a failure. Lets inspect the container.

```
kubectl describe pod nginx1401 -n dev1401
```

`
 Warning  Unhealthy  56s (x30 over 5m46s)  kubelet, node01    Readiness probe failed: Get "http://10.244.1.2:8080/": dial tcp 10.244.1.2:8080: connect: connection refused
 `

