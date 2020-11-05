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