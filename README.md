# Lab1 - Learning Kubernetes

List existing pods: `kubectl get pods`

Create nginx pod: `kubectl run nginx --image=nginx `
Check again the existing pods: `kubectl get pods`
Describe a pods: `kubectl describe pods newpods-vr8pd`

Describe a node to see what is running

```
kubectl describe nodes node01
kubectl describe nodes master
```

Also we can check where is running the container: `kubectl get pods -o wide`

When we run `kubectl get pods` the column READY is made up of #ContainersRunning/#TotalContainers

Delete a pod

```
kubectl get all
kubectl delete name   --- this name is from the column prev cmd
```

Another option is `kubectl delete pod pod_name`


Create a yaml file for nginx:

```
apiVersion: v1

kind: Pod

metadata:
    name: nginx-app

spec:
    containers:
        - name: nginx
          image: nginx
          ports:
              - containerPort: 80
```


Load the pod from the yaml file:

```
kubectl apply -f filename.yml
kubectl get pods
```

also can be created using: `kubectl create -f file_name.yml`

if image is wrong, you can edit usit the following cmd `kubectl edit pod/redis`

Check events in kubernetes `kubectl get events`

We can extract the yaml file using the following cmd: `kubectl get pod pod_name -o yaml > pod_definition.yaml`


## Help Commands

Print all the different resources: `kubectl api-resources`

Print the different valid apis: `kubectl api-versions`

Print valid sections for an element: `kubectl explain services --recursive` Replace services for the element u're interested in.

Also we can print info about any of the sections of an object. `kubectl explain services.spec` Replace services and spec for the object and section u're inrested in
respectively.

We can add the nivel of granularity, adding the key that we're interested in such as `kubectl explain services.spec.type`

## How to check changes to be applied

Show what will do the yaml file `kubectl apply -f fileName.yml --dry-run`
Resources to be created `kubectl apply -f fileName.yml --server-dry-run`
Check differences `kubectl diff -f fileName.yml`

