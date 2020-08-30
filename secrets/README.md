# Secrets

https://kubernetes.io/docs/concepts/configuration/secret/

## 1. How many secrets exists on the system

`kubectl get secrets`

## 2. Which of the following is not a secret data defined in 'default-token' secret?

* type
* token
* namespace
* ca.crt

`kubectl describe secret default-token-pkll7`

## 3. Create a secret with the following patameters:

* Secret name: db-secret
* secret 1: DB_Host=sql01
* secret 2 DB_User root
* secret 3 DB_Password password123

`kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`


## 4. Configure webapp-pod to load environment variables from the nearly created secret. Delete and recreate the pod if required

* Pod name: webapp-pod
* Image name: kodekloud/simple-webapp-mysql
* Env From: Secret=db-secret

```
kubectl get pod webapp-pod -o=yaml > webapp.yaml
kubectl delete pod webapp-pod
cat webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret

kubectl apply -f output.yaml
```