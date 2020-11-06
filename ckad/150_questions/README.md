# Practice enough with these questions  for the CKAD exam

https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552

## Core Concepts (13%)

https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-
https://kubernetes.io/docs/concepts/workloads/pods/
https://kubernetes.io/docs/reference/kubectl/jsonpath/

### 1. List all the namespaces in the cluster
```
kubectl get ns
```

### 2. List all the pods in all namespaces
```
kubectl get pods --all-namespaces
```

### 3. List all the pods in the particular namespace
```
kubectl get pods -n kube-system
```

### 4. List all the services in the particular namespace
```
kubectl get svc -n kube-system
```

### 5. List all the pods showing name and namespace with a json path expression
```
kubectl explain pods.metadata
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"
```

### 6. Create an nginx pod in a default namespace and verify the pod running
```
kubectl run nginx --image=nginx --restart=Never
kubectl get pods -l 'run=nginx'
```

### 7. Create the same nginx pod with a yaml file
```
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > output.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```
kubectl create -f output.yaml
kubectl get pods -l 'run=nginx'
```

### 8. Output the yaml file of the pod you just created
```
kubectl get pod nginx -o yaml
```

### 9. Get the complete details of the pod you just created
```
kubectl describe pod nginx
```

### 10. Delete the pod you just created
```
kubectl delete -f output.yaml
```

or

```
kubectl delete pod nginx
```

### 11. Delete the pod you just created without any delay (force delete)
```
kubectl delete pod nginx --force
```

### 12. Create the nginx pod with version 1.17.4 and expose it on port 80
```
kubectl run nginx --image=nginx:1.17.4 --port=80
```

### 13. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated
```
kubectl edit pod nginx
kubectl describe pod nginx
```

### 14. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes
```
kubectl edit pod nginx
kubectl describe pod nginx
kubectl get pod nginx -w
```

### 15. Check the Image version without the describe command
```
kubectl explain pods.spec.containers
kubectl get pod nginx -o jsonpath="{.spec.containers[].image}" | awk -F ":" '{ print $2 }'
```

### 16. Create the nginx pod and execute the simple shell on the pod
```
kubectl run nginx --image=nginx --restart=Never
kubectl exec -it nginx -- /bin/sh
```

### 17. Get the IP Address of the pod you just created
```
get pod nginx  -o wide --no-headers
```

### 18. Create a busybox pod and run command ls while creating it and check the logs
```
kubectl run busybox --image=busybox -- ls
kubectl logs busybox
```

### 19. If pod crashed check the previous logs of the pod
```
kubectl logs --help
kubectl logs busybox -p
```

### 20. Create a busybox pod with command sleep 3600
```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'sleep 3600'
```

### 21. Check the connection of the nginx pod from the busybox pod
```
kubectl get pods -o wide
kubectl exec -it busybox -- /bin/sh
wget -O- 10.1.0.108:80
```

### 22. Create a busybox pod and echo message ‘How are you’ and delete it manually
```
kubectl run busybox --image=nginx --restart=Never -it -- echo "How are you"
kubectl delete pod busybox
```

### 23. Create a busybox pod and echo message ‘How are you’ and have it deleted immediately
```
kubectl run busybox --image=nginx --restart=Never --rm -it -- echo "How are you"
```

### 24. Create an nginx pod and list the pod with different levels of verbosity
```
kubectl run nginx --image=nginx --restart=Never --port=80
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9
``` 

### 25. List the nginx pod with custom columns POD_NAME and POD_STATUS
```
kubectl get pods --help # look for custom columns
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
```

### 26. List all the pods sorted by name

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

```
kubectl explain pod.metadata
kubectl get pods --sort-by=.metadata.name
```

### 27. List all the pods sorted by created timestamp
```
kubectl explain pod.metadata
kubectl get pods --sort-by=.metadata.creationTimestamp
```