# Resource creation

## 1. How many nodes does your cluster have?

```
kubectl get nodes --no-headers | wc -l
```

## 2. What kernel version and what container engine is each node running?

```
kubectl describe node docker-desktop
```

## 3. List only the pods in the kube-system namespace.

```
kubectl get pods --namespace=kube-system
```

## 4. Explain the role of some of these pods.

* **etcd**: database managing cluster state
* **apiserver**: restapi handling incoming and outgoing communication of user requests and kubelet requests and responses
* **coredns**: name resolution for network communications   

They're pods to manage the system

## 5. Create a deployment using kubectl create that runs the image bretfisher/clock and name it ticktock.

```
kubectl create deployment ticktock --image=bretfisher/clock
```

## 6. Increase the number of pods running in that deployment to three.

```
kubectl scale deployment.v1.apps/ticktock --replicas=3
kubectl scale deployment ticktock --replicas=3
```

## 7. Use a selector to output only the last line of logs of each container.

```
kubectl logs -l app=ticktock --tail 1  
kubectl logs --selector=app=ticktock --tail=1
```