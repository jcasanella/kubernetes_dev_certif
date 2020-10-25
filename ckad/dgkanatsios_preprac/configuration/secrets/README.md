# Secrets

https://kubernetes.io/docs/concepts/configuration/secret/
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

## 1. Create a secret called mysecret with the values password=mypass

```
kubectl create secret generic mysecret --from-literal=password=mypass
kubectl get secrets
```

## 2. Create a secret called mysecret2 that gets key/value from a file

```
echo -n 'admin' > ./username.txt
kubectl create secret generic mysecret2 --from-file=username.txt
kubectl get secrets
```

## 3. Get the value of mysecret2

```
kubectl get secret mysecret2 -o jsonpath='{.data}' # return something like map[username.txt:YWRtaW4=]
echo 'YWRtaW4=' | base64 --decode
```

## 4. Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Add the following sections: `volumeMounts` and `volumes`

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
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret2
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl apply -f output.yaml
kubectl exec -it nginx -- /bin/bash
ls /etc/foo
cat /etc/foo/username.txt
```

## 5. Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

```
kubectl delete pod nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Add section `env`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    env:
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret2
            key: username.txt
  restartPolicy: Never
```

```
kubectl apply -f output.yaml
kubectl exec -it nginx -- env | grep USERNAME
```

**Note**: The key is the name used in the filename to create the secret
