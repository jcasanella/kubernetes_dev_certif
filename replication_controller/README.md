# ReplicationController

Create the yml file that defines the ReplicationController. `kubectl create -f rc-definition.yml`

To check the ReplicationController: `kubectl get replicationcontroller`

If we check the pods created will see something similar to this:

```
$ kubectl get pods                                                                     

NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-hlpkj   1/1     Running   0          14s
myapp-rc-jl8tl   1/1     Running   0          14s
myapp-rc-srkfc   1/1     Running   0          14s
```

Drop ReplicationController: `kubectl delete ReplicationController myapp-rc`

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
