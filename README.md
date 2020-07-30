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



