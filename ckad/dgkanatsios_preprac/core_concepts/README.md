# Core Concepts (13%)

References:

* kubernetes > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

* kubernetes > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters] (https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters using API](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

## Exercises

### 1. Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

```
kubectl create ns mynamespace
kubectl get ns --no-headers
kubectl run nginx --image nginx --namespace mynamespace
kubectl get pods --namespace=mynamespace
```

### 2. Create the pod that was just described using YAML

```
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - name: nginx
    image: nginx
```

```
kubectl create -f pod.yaml
kubectl create -f pod.yaml -n mynamespace
```

### 3. Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

```
kubectl run busybox --image=busybox -- env
kubectl logs busybox
```

## 4. Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

```
kubectl run busybox --image busybox --dry-run=client -o yaml > pod.yaml    
```  

Add `command: ["env"]` to the pod definition

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
    resources: {}
    command: ["env"]
  restartPolicy: Always
  dnsPolicy: ClusterFirst
status: {}
```
```
kubectl create -f pod.yaml
```

**Note**: We can do everything in one line with the following command: `kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml`

## 5. Get the YAML for a new namespace called 'myns' without creating it

```
kubectl create ns myns --dry-run=client -o yaml > myns.yaml
```

## 6. Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

https://kubernetes.io/docs/concepts/policy/resource-quotas/

A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.

```
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myrq
spec:
  hard:
    cpu: "1"
    memory: 1G
    pods: "2"
```