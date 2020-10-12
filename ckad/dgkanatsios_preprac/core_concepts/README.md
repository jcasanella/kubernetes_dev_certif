# Core Concepts (13%)

References:

* kubernetes > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

* kubernetes > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters] (https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters using API](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)

* kubernetes > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

## Some theory

The main components are running in the `kube-system` namespace

* **kubernetes proxy**: responsible for routing network traffic to load-balanced services in the Kubernetes cluster must be present on every node in the cluster.

* **kubernetes dns**: runs a DNS server, which provides naming and discovery for the services that are defined in the cluster

* **Kubernetes UI**: The UI is run as a single replica, but it is still managed by a Kubernetes deployment for reliability and upgrades

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

**Note**: `kubectl logs -f pood_name` will continuosly streaming logs. Adding the `--previous` flag will get logs from a previous instance of the container.

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

## 7. Get pods on all namespaces

```
kubectl get pods --all-namespaces
```

## 8. Create a pod with image nginx called nginx and expose traffic on port 80

```
kubectl run nginx --image=nginx --port=80
```

## 9. Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled

```
kubectl get pods
kubectl edit pod nginx
```

Replace `image: nginx` for `image: nginx:1.7.1`

```
kubectl describe pod nginx 
```

## 10. Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

```
kubectl get po -o wide                  # this will give you the ip of the pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

**Note**: wget -O- what is doing is ` -O FILE         Save to FILE ('-' for stdout)`

## 11. Get pod's YAML

```
kubectl get pod nginx -o yaml
```

## 12. Get information about the pod, including details about potential issues (e.g. pod hasn't started)

```
kubectl describe pod nginx
```

## 13. Get pod logs

```
kubectl logs -h                     # to get some help
kubectl logs nginx
```

## 14. If pod crashed and restarted, get logs about the previous instance

If we check the help from logs, `kubectl logs -h` will see which parameter we need to use

```
kubectl logs -p=true nginx
```

## 15. Execute a simple shell on the nginx pod

```
kubectl exec -it nginx -- bash 
```

## 16. Create a busybox pod that echoes 'hello world' and then exits

**Note**: if u dont remember how to create a temporal pod, check the documentation `kubectl run -h`

```
kubectl run busybox --image busybox -it --restart=Never -- echo "hello world"
```

## 17. Do the same, but have the pod deleted automatically when it's completed

```
kubectl run busybox --image busybox --rm -it --restart=Never -- echo "hello world"
```

## 18. Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

```
kubectl run nginx --image=nginx --env="var1=val1"
kubectl exec -t nginx -- env  
kubectl exec -t nginx -- sh -c 'echo ${var1}'        
```