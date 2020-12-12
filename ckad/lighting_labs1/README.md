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
    path: /opt/volume/nginx
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
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 200Mi
  storageClassName: manual
```
```
kubectl apply -f pvc.yaml
kubectl get pvc
```
```
kubectl run logger --image=nginx:alpine --dry-run=client -o yaml > output.yaml
```

Create the pod with the volumne mounted:

```
apiVersion: v1
kind: Pod
metadata:
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
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```
kubectl create -f output.yaml
```

## 2. We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working. Make sure that incoming connection from the pod webapp-color are successful.

```
kubectl get pods
kubectl get service secure-service
```

NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
secure-service   ClusterIP   10.98.94.218   <none>        80/TCP    3m1s

Expecting to run on port `80`, check if pod is running on port `80`

```
kubectl exec -it webapp-color -- sh
nc -z secure-service 80 (can not connect)
nc -z -w 1 secure-service 80
```

Check policies:

```
kubectl get netpol
kubectl describe netpol default-deny
kubectl get netpol default-deny -o yaml > netpol.yaml
```

Original file:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: "2020-12-12T16:33:00Z"
  generation: 1
  managedFields:
  - apiVersion: networking.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:policyTypes: {}
    manager: kubectl-create
    operation: Update
    time: "2020-12-12T16:33:00Z"
  name: default-deny
  namespace: default
  resourceVersion: "1885"
  selfLink: /apis/networking.k8s.io/v1/namespaces/default/networkpolicies/default-deny
  uid: 58e73a4c-0c42-4219-a747-b1463a3c3aac
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ```


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

After changes:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
spec:
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

```
kubectl apply -f policy.yaml
```

```
kubectl exec -it webapp-color -- sh
nc -z secure-service 80 
```

## 3. Create a pod called time-check in the dvl1987 namespace. This pod should run a container called time-check that uses the busybox image. 

* Create a config map called time-config with the data TIME_FREQ=10 in the same namespace. 
* The time-check container should run the command: while true; do date; sleep $TIME_FREQ;done and write the result to the location /opt/time/time-check.log. 
* The path /opt/time on the pod should mount a volume that lasts the lifetime of this pod.

Check if the keyspace exists

```
kubectl get ns
kubectl create ns dvl1987
```

Create a template for the podL

```
kubectl run time-check --image=busybox -n dvl1987 --dry-run=client -o yaml -- /bin/sh -c "while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log" > output.yaml
```

Create the Config-map:
```
kubectl create configmap time-config --from-literal=TIME_FREQ=10 -n dvl1987
```

Edit the pod definition in oder to add the volume and the configmap

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do date; sleep ; done > /opt/time/time-check.log
    image: busybox
    name: time-check
    resources: {}
    volumeMounts:
    - mountPath: /opt/time/
      name: mydir
    env:
      - name: TIME_FREQ
        valueFrom:
          configMapKeyRef:
            name: time-config
            key: TIME_FREQ
  volumes:
  - name: mydir
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl apply -f output.yaml
kubectl get pods -n dvl1987
```

## 4. Create a new deployment called nginx-deploy, with one single container called nginx, image nginx:1.16 and 4 replicas. 

* The deployment should use RollingUpdate strategy with **maxSurge=1**, and **maxUnavailable=2**.
* Next upgrade the deployment to version 1.17 using rolling update.
* Finally, once all pods are updated, undo the update and go back to the previous version.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 4
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
```
```
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl describe deployment nginx-deploy
```
```
kubectl set image deployment/nginx-deploy nginx=nginx:1.17
kubectl rollout undo deployment/nginx-deploy
```

## 5. Create a redis deployment with the following parameters:

* Name of the deployment should be redis using the redis:alpine image. It should have exactly 1 replica.
* The container should request for .2 CPU. It should use the label app=redis.
* It should mount exactly 2 volumes:
* Make sure that the pod is scheduled on master/controlplane node.
* An Empty directory volume called data at path /redis-master-data.
* A configmap volume called redis-config at path /redis-master.
* The container should expose the port 6379.

```
kubectl create deployment redis --image=redis:alpine --dry-run=client -o yaml > output.yaml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      nodeName: master
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: redis-config
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
```
```
kubectl apply -f output.yaml
```
