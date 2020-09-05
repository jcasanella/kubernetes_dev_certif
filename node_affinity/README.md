# Node Affinity

https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity

## 1. How many Labels exist on node node01?

`kubectl describe node node01 | grep 'Labels' -A 15`

also we can get the labels with the following command: `kubectl get nodes node01 --show-labels`

## 2. Apply a label color=blue to node node01

`kubectl edit node node01`

## 3. Create a new deployment named 'blue' with the NGINX image and 6 replicas

* Name: blue
* Replicas: 6
* Image: nginx

```
kubectl create deployment blue --image=nginx --dry-run -o yaml > output.yaml
```

change the number of replicas to 6 and apply the changes. `kubectl apply -f output.yaml`

Other option: 

```
kubectl label nodes node01 color=blue
kubectl create deployment blue --image=nginx
kubectl scale deployment blue --replicas=6
```

## 4. Which nodes can the PODs placed on?

Check if master and node01 have any taints on them that will prevent the pods to be scheduled on them. If there are no taints, the pods can be scheduled on either node.

```
kubectl describe node node01 | grep 'Taints' -A 5
kubectl describe node controlplane | grep 'Taints' -A 5
```

## 5. Set Node Affinity to the deployment to place the PODs on node01 only

* Name: blue
* Replicas: 6
* Image: nginx
* NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
* Key: color
* values: blue

Add the affinity to the deployment

```
kubectl edit deployment blue

    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
    containers:
    - image: nginx
    ....      
```

## 6. Create a new deployment named 'red' with the NGINX image and 3 replicas, and ensure it gets placed on the master node only. Use the label - node-role.kubernetes.io/master - set on the master node.

* Name: red
* Replicas: 3
* Image: nginx
* NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
* Key: node-role.kubernetes.io/master
* Use the right operator

kubectl create deployment red --image=nginx --dry-run=server -o yaml > output.yaml

replace replicas from 1 to 3 -- look entry replicas

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists

kubectl apply -f output.yaml

 