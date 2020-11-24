# Lightning Lab1

## Exercise 1

* Create a **Persistent Volume** called **log-volume**. It should make use of a **storage class** name **manual**. It should use RWX as the access mode and have a size of 1Gi. The volume should use the **hostPath** /opt/volume/nginx
* Next, create a **PVC** called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.
* Mount this in a pod called **logger** at the location /var/www/nginx. This pod should use the image nginx:alpine.

Lets start creating the `PV`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: manual
  hostPath:
    path: /opt/volumne/nginx
```
```
kubectl create -f pv.yaml
```

Next step  is to ask for the `PVC`

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```

```
kubectl run logger --image=nginx:alpine --dry-run=client -o yaml > output.yaml
```

Create the pod with the volumne mounted:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: logger
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    resources: {}
    volumeMounts:
    - mountPath: "/var/www/nginx"
      name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: log-claim
status: {}
```
```
kubectl create -f output.yaml
```

## 2. We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
Troubleshoot why this is happening.

Make sure that incoming connection from the pod webapp-color are successful.


Important: Don't delete any current objects deployed.


kubectl get pods
kubectl get service secure-service

NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
secure-service   ClusterIP   10.98.94.218   <none>        80/TCP    3m1s

Expecting to run on port 89, check if pod is running on port 80

kubectl exec -it webapp-color -- sh
nc -z secure-service 80 (can not connect)
nc -z -w 1 secure-service 80

Check policies:

kubectl get netpol
kubectl describe netpol default-deny
kubectl get netpol default-deny -o yaml > netpol.yaml

Change name of the netpolicy and in spec

```
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

kubectl apply -f policy.yaml