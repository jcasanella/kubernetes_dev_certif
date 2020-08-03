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
