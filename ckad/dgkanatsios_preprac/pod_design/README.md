# Pod Design

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

**Annotations**:

You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata

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