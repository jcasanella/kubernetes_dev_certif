# Pod Design

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

**Annotations**:

You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata

**Resources**:

Resources are requested per container, not per Pod. The total
resources requested by the Pod is the sum of all resources requested
by all containers in the Pod. The reason for this is that in many
cases the different containers have very different CPU requirements.
For example, in the web server and data synchronizer Pod,
the web server is user-facing and likely needs a great deal of CPU,
while the data synchronizer can make do with very little.

With Kubernetes, a Pod requests the resources required to run its containers. Kubernetes
guarantees that these resources are available to the Pod. The most commonly requested resources are CPU and memory, but Kubernetes has support for other
resource types as well, such as GPUs and more

```
apiVersion: v1
  kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

The Kubernetes scheduler will
ensure that the sum of all requests of all Pods on a node does not exceed the capacity
of the node. Therefore, a Pod is guaranteed to have at least the requested resources
when running on the node.

In addition to setting the resources required by a Pod, which establishes the minimum
resources available to the Pod, you can also set a maximum on a Pod’s resource
usage via resource limits.

```
apiVersion: v1
  kind: Pod
metadata:
  name: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
```

## Exercises

## 1. Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

```
kubectl run nginx1 --image=nginx --labels="app=v1"
kubectl run nginx2 --image=nginx --labels="app=v1"
kubectl run nginx3 --image=nginx --labels="app=v1"
```

Same but using a bash script

```
#!/bin/bash

for i in 1 2 3
do
    cmd="kubectl run nginx${i} --image=nginx --labels=\"app=v1\""
    echo ${cmd}
    eval ${cmd}
done
```

## 2. Show all labels of the pods

```
kubectl get pods --show-labels   
```

## 3. Change the labels of pod 'nginx2' to be app=v2

```
kubectl edit pod nginx2 
```

and change the app=v1 to `app: v2`

```
kubectl get pods --show-labels  
```

**Note**: To replace in vi `:%s/FindMe/ReplaceME/g`

other way to do the same:

```
kubectl label po nginx2 app=v2 --overwrite
```

## 4. Get the label 'app' for the pods

```
kubectl get pods --show-labels  | grep app 
```

```
kubectl get pods -l app
``` 
or 
```
kubectl get po --label-columns=app
```

## 5. Get only the 'app=v2' pods

```
kubectl get pods -l app=v2
```

## 6. Remove the 'app' label from the pods we created before

**Note**: We can check label help with `kubectl labels --help`

```
kubectl label pods --all app-
kubectl get pods --show-labels
```

Other option is:

```
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -l app app-
```

## 7. Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

```
kubectl get nodes
kubectl label node kind-worker accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: "nginx"
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

```
kubectl get pods -o wide
```

## 8. Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

Create pods:

```
#!/bin/bash

for i in 1 2 3
do 
    cmd="kubectl run nginx${i} --image=nginx"
    eval ${cmd}
done
```

Add annotations

```
kubectl annotate pods nginx{1..3} description='my description'  
```

## 9. Check the annotations for pod nginx1

```
kubectl describe pod nginx1 | grep -i annotat
```

## 10. Remove the annotations for these three pods

```
kubectl annotate pods nginx{1..3} description-
```

## 11. Remove these pods to have a clean state in your cluster

```
kubectl delete pod nginx{1..3}
```