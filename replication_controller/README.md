# ReplicationController

https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

Main difference between ReplicationController and Replicaset. The Replicaset is the new generation and allows to specify more complex selectors. For replica sets, filtering is done according to a set of values. The supported operators are *in*, *notin*, and *exists* (only for the key). For example, a replication controller can select pods such as environment = dev. A replica set can select pods such as environment
in ["dev", "test"].

Replica sets are generally never created on their own. Deployments own and manage replica sets to orchestrate pod creation, deletion and updates.

## 1. Create the yml file that defines the ReplicationController. 

`kubectl create -f rc-definition.yml`

To check the ReplicationController: `kubectl get replicationcontroller`

If we check the pods created will see something similar to this:

```
$ kubectl get pods                                                                     

NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-hlpkj   1/1     Running   0          14s
myapp-rc-jl8tl   1/1     Running   0          14s
myapp-rc-srkfc   1/1     Running   0          14s
```

## 2. Drop ReplicationController: 

`kubectl delete ReplicationController myapp-rc`

How to check replicaset: 
```
kubectl get rs
kubectl get replicasets
```

To describe a replicaset: `kubectl describe replicaset` It will show all info about this replicaset
Get the yaml from a replicaset: `kubectl get rs name-replica-set -o yaml > output_file.yaml`

Also from the description of the replicaset we can see the desired pods and how many of them are running.

If we need to change the existing settings for a replicaset: `kubectl edit rs name-replicaset`
When the change is related to a pod, this is not applied until the pod is recreated again. Tip. u can delete the pod to force to be recreated again

Example of Replicaset

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: wildfly-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wildfly-rs-pod
    matchExpressions:
      - {key: tier, operator: In, values: ["backend"]}
      - {key: environment, operator: NotIn, values: ["prod"]}
template:
  metadata:
    labels:
      app: wildfly-rs-pod
      tier: backend
      environment: dev
spec:
  containers:
  - name: wildfly
    image: jboss/wildfly:10.1.0.Final
    ports:
    - containerPort: 8080
```
