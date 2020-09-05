# Taint - toleration

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

## 1. How many Nodes exist on the system? including the master node

`kubectl get nodes`

## 2. Do any taints exist on node01?

`kubectl describe node node01 | grep 'Taints'`

## 3.Create a taint on node01 with key of 'spray', value of 'mortein' and effect of 'NoSchedule'

* Key = spray
* Value = mortein
* Effect = NoSchedule

```
kubectl taint nodes node01 spray=mortein:NoSchedule
kubectl describe node node01 | grep 'Taints'
```

## 4. Create a new pod with the NGINX image, and Pod name as 'mosquito'

```
kubectl run mosquito --image=nginx
kubectl get pods | grep 'mosquito'
```

## 5. Create another pod named 'bee' with the NGINX image, which has a toleration set to the taint Mortein

* Image name: nginx
* Key: spray
* Value: mortein
* Effect: NoSchedule
* Status: Running

Remember we can create the yaml file with `kubectl run bee --image=nginx --dry-run -o yaml > output.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - name: bee
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"  

kubectl apply -f output.yaml
```

## 6. Notice the 'bee' pod was scheduled on node node01 despite the taint.

```
kubectl get pods -o wide

NAME       READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          55s   10.244.1.4   node01   <none>           <none>
mosquito   0/1     Pending   0          12m   <none>       <none>   <none>           <none>
```

## 7. Do you see any taints on master node?

```
kubectl describe node master | grep 'Taints'
```

## 8. Remove the taint on master, which currently has the taint effect of NoSchedule

`kubectl edit node master`

look for taints property and remove it.

## 9. Which node is the POD 'mosquito' on now?

`kubectl get pods -o wide`
