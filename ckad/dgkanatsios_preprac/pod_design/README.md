# Pod Design

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

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