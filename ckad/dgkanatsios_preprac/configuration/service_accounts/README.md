# Service Accounts

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

## 1. See all the service accounts of the cluster in all namespaces

```
kubectl get sa --all-namespaces
```

## 2. Create a new serviceaccount called 'myuser'

```
kubectl create sa myuser
kubectl get sa
```

## 3. Create an nginx pod that uses 'myuser' as a service account

```
kubectl run nginx --image=nginx --dry-run=client --serviceaccount='myuser' -o yaml > output.yaml
kubectl describe pod nginx
```

Look for this entry in the describe output: ```SecretName:  myuser-token-`