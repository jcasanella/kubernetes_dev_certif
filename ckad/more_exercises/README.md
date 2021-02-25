## 1. Creating pods

### 1. Create a pod of image bash which runs once to execute the `command hostname > /tmp/hostname && sleep 1d`

```
 $ kubectl run --image=bash -- bash -c 'hostname > /tmp/hostname && sleep 1d'

 $ kubectl exec -it bash -- bash -c 'cat /tmp/hostname'
 bash
 ```

 ### 2. Export and edit its YAML to add a label `my-label: test`

```
$ kubectl get pod bash -o yaml > bash_pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-02-22T20:22:01Z"
  labels:
    run: bash
    my-label: test ## Line added
  name: bash
  namespace: default
spec:
  containers:
  - args:
    - -c
    - hostname > /tmp/hostname && sleep 1d
    image: bash
    imagePullPolicy: Always
    name: bash
    resources: {}
```

### 3. Create the pod from the YAML file.

```
$ kubectl apply -f bash_pod.yaml
$ kubectl get pods --show-labels

bash    1/1     Running   0          12m     my-label=test,run=bash
nginx   1/1     Running   0          4d20h   run=nginx
```

### 4. Delete the pod instantly without waiting time

```
$ kubectl delete pod bash --grace-period=0 --force
$ kubectl get pods

NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          4d20h
```

## 2. Namespaces, deployments and services

### 1. Create a new namespace `k8s-challenge-2-a` and assure all following operations (unless different namespace is mentioned) are done in this namespace.

```
$ kubectl create ns k8s-challenge-2-a
$ kubectl get ns

NAME                STATUS   AGE
default             Active   15d
k8s-challenge-2-a   Active   16s
kube-node-lease     Active   15d
kube-public         Active   15d
kube-system         Active   15d
```

### 2. Create a deployment named nginx-deployment of three pods running image nginx with a memory limit of 64MB.

```
$ kubectl create deployment nginx-deployment --replicas=3 --image=nginx --dry-run=client -o yaml -n k8s-challenge-2-a > deployment2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:          # Added
          limits:           # Added
            memory: "64Mi"  # Added
status: {}

$ kubectl apply -f deployment2.yaml
$ kubectl get deployments -n k8s-challenge-2-a

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           9s

$ kubectl get pods -n k8s-challenge-2-a

NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-69f87cb4c-254xv   1/1     Running   0          48s
nginx-deployment-69f87cb4c-mpvx5   1/1     Running   0          48s
nginx-deployment-69f87cb4c-vdfrb   1/1     Running   0          48s
```

### 3. Expose this deployment under the name `nginx-service` inside our cluster on port `4444`, so point the service port `4444` to pod ports `80`.
```
$ kubectl expose deployment nginx-deployment --port=4444 --target-port=80 --name=nginx-service -n k8s-challenge-2-a
$ kubectl get services -n k8s-challenge-2-a

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
nginx-service   ClusterIP   10.97.196.69   <none>        4444/TCP   25s

$ kubectl describe service nginx-service -n k8s-challenge-2-a

Name:              nginx-service
Namespace:         k8s-challenge-2-a
Labels:            app=nginx-deployment
Annotations:       <none>
Selector:          app=nginx-deployment
Type:              ClusterIP
IP Families:       <none>
IP:                10.97.196.69
IPs:               <none>
Port:              <unset>  4444/TCP
TargetPort:        80/TCP
Endpoints:         10.1.1.181:80,10.1.1.182:80,10.1.1.183:80
Session Affinity:  None
Events:            <none>
```

### 4. Spin up a temporary pod with image `cosmintitei/bash-curl`, ssh into it and request the default nginx page on port `4444` of our `nginx-service` using `curl`.

```
$ kubectl run -it bash --image=cosmintitei/bash-curl -n k8s-challenge-2-a -- bash
$ curl nginx-service:4444

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 5. Spin up a temporary with image `cosmintitei/bash-curl` in `default` namespace, ssh into it and request the default nginx page on port `4444` of our `nginx-service` in namespace `k8s-challenge-2-a` using curl.

```
$ kubectl run -it bash --image=cosmintitei/bash-curl -- bash
$ curl nginx-service:4444
curl: (6) Couldn't resolve host 'nginx-service'
```

## 3. Cronjobs and Volumes

### 1. Create a static PersistentVolume of `50MB` to your `nodes/localhosts` `/tmp/k8s-challenge-3` directory.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-task
spec:
  storageClassName: manual
  capacity:
    storage: 50Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /tmp/k8s-challenge-3

$ kubectl apply -f pv.yaml
$ kubectl get pv
```

### 2. Create a PersistentVolumeClaim for this volume for `40MB`.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi
  storageClassName: manual

$ kubectl apply -f pvc.yaml
$ kubectl get pvc

NAME      STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myclaim   Bound    pv-task   50Mi       RWO            manual         3m41s
```


### 3. Create a CronJob which runs two instances every minute of a pod mounting the PersistentStorageClaim into `/tmp/vol` and executing the command `hostname >> /tmp/vol/storage`.
```
$ kubectl create cronjob busybox --image=busybox  --schedule="*/1 * * * *" --dry-run=client -o yaml > output3.yaml

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busybox
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busybox
    spec:
      parallelism: 2
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: busybox
            args:           # added
            - /bin/sh       # added
            - -c            # added
            - hostname >> /tmp/vol/stotage # added
            resources: {}
            volumeMounts:   # added
            - mountPath: "/tmp/vol" # added
              name: mypd    # added
            volumeMounts:   # added
            - mountPath: "/tmp/vol" # added
              name: mypd    # added
          restartPolicy: OnFailure
          volumes:    # added
          - name: mypd  # added
            persistentVolumeClaim:  # added
              claimName: myclaim    # added
  schedule: '*/1 * * * *'
status: {}

$ kubectl apply -f output3.yaml
$ kubectl get jobs

NAME                 COMPLETIONS   DURATION   AGE
busybox-1614198300   1/1           5s         9s
```

## 4. We only need to keep the last 4 successful executed jobs in the cronjobs history.

Add to the yaml created in the previous exercise:

`successfulJobsHistoryLimit: 4`

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busybox
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busybox
    spec:
      parallelism: 2
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: busybox
            args:           
            - /bin/sh       
            - -c            
            - hostname >> /tmp/vol/stotage 
            resources: {}
            volumeMounts:   # added
            - mountPath: "/tmp/vol" 
              name: mypd    # added
            volumeMounts:   # added
            - mountPath: "/tmp/vol" 
              name: mypd    # added
          restartPolicy: OnFailure
          volumes:    # added
          - name: mypd  # added
            persistentVolumeClaim:  
              claimName: myclaim    
  schedule: '*/1 * * * *'
  successfulJobsHistoryLimit: 4 # Added
status: {}
```

