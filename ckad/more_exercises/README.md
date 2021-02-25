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

### 4. We only need to keep the last 4 successful executed jobs in the cronjobs history.

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

## 4. Deployment Rollout Rollback

### 1. Create a deployment of 15 pods with image nginx:1.14.2 in namespace one (also create that namespace).
```
$ kubectl create ns one
$ kubectl create deployment nginxdeployment --image=nginx:1.14.2 --namespace=one --replicas=15
$ kubectl get deployments --namespace=one

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginxdeployment   15/15   15           15          42s

$ kubectl get pods --namespace=one

NAME                              READY   STATUS    RESTARTS   AGE
nginxdeployment-c8ffd4c79-47v9k   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-7b249   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-8vs7r   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-8xkpx   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-9g6m2   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-9pdnf   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-c48bq   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-drldf   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-gjbh4   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-k2qrg   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-lbd99   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-qm98t   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-sgsqj   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-tlpx2   1/1     Running   0          61s
nginxdeployment-c8ffd4c79-znjpr   1/1     Running   0          61s
```

### 2. Confirm that all pods are running that image.
```
$ kubectl describe deployment nginxdeployment --namespace=one

Name:                   nginxdeployment
Namespace:              one
CreationTimestamp:      Thu, 25 Feb 2021 23:17:51 +0000
Labels:                 app=nginxdeployment
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginxdeployment
Replicas:               15 desired | 15 updated | 15 total | 15 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginxdeployment
  Containers:
   nginx:
    Image:        nginx:1.14.2
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginxdeployment-c8ffd4c79 (15/15 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m52s  deployment-controller  Scaled up replica set nginxdeployment-c8ffd4c79 to 15
```

### 3. Edit the deployment to change the image of all pods to nginx:1.15.10.
```
$ kubectl set image deployment/nginxdeployment nginx=nginx:1.15.10 --namespace=one

$ kubectl describe deployment nginxdeployment --namespace=one

Name:                   nginxdeployment
Namespace:              one
CreationTimestamp:      Thu, 25 Feb 2021 23:17:51 +0000
Labels:                 app=nginxdeployment
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginxdeployment
Replicas:               15 desired | 15 updated | 15 total | 15 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginxdeployment
  Containers:
   nginx:
    Image:        nginx:1.15.10
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginxdeployment-7cbc44bb97 (15/15 replicas created)
Events:
  Type    Reason             Age                 From                   Message
  ----    ------             ----                ----                   -------
  Normal  ScalingReplicaSet  6m42s               deployment-controller  Scaled up replica set nginxdeployment-c8ffd4c79 to 15
  Normal  ScalingReplicaSet  32s                 deployment-controller  Scaled up replica set nginxdeployment-7cbc44bb97 to 4
  Normal  ScalingReplicaSet  32s                 deployment-controller  Scaled down replica set nginxdeployment-c8ffd4c79 to 12
  Normal  ScalingReplicaSet  32s                 deployment-controller  Scaled up replica set nginxdeployment-7cbc44bb97 to 7
  Normal  ScalingReplicaSet  24s                 deployment-controller  Scaled down replica set nginxdeployment-c8ffd4c79 to 11
  Normal  ScalingReplicaSet  24s                 deployment-controller  Scaled up replica set nginxdeployment-7cbc44bb97 to 8
  Normal  ScalingReplicaSet  23s                 deployment-controller  Scaled down replica set nginxdeployment-c8ffd4c79 to 10
  Normal  ScalingReplicaSet  23s                 deployment-controller  Scaled up replica set nginxdeployment-7cbc44bb97 to 9
  Normal  ScalingReplicaSet  23s                 deployment-controller  Scaled down replica set nginxdeployment-c8ffd4c79 to 9
  Normal  ScalingReplicaSet  13s (x15 over 23s)  deployment-controller  (combined from similar events): Scaled down replica set nginxdeployment-c8ffd4c79 to 0

$ kubectl get pods --namespace=one -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image
CONTAINER   IMAGE
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
```

### 4. Edit the deployment to change image of all pods to `nginx:1.15.666`.
```
$ kubectl set image deployment/nginxdeployment nginx=nginx:1.15.666 --namespace=one
$ kubectl get pods --namespace=one

NAME                               READY   STATUS             RESTARTS   AGE
nginxdeployment-5cf4dfd5b8-8lvtc   0/1     ErrImagePull       0          81s
nginxdeployment-5cf4dfd5b8-946px   0/1     ImagePullBackOff   0          81s
nginxdeployment-5cf4dfd5b8-gn2hk   0/1     ErrImagePull       0          81s
nginxdeployment-5cf4dfd5b8-xw6kh   0/1     ImagePullBackOff   0          81s
nginxdeployment-5cf4dfd5b8-z9r8w   0/1     ImagePullBackOff   0          81s
nginxdeployment-5cf4dfd5b8-zd9b9   0/1     ImagePullBackOff   0          81s
nginxdeployment-5cf4dfd5b8-zn7xz   0/1     ErrImagePull       0          81s

$ kubectl describe pod nginxdeployment-5cf4dfd5b8-zn7xz --namespace=one

Name:         nginxdeployment-5cf4dfd5b8-zn7xz
Namespace:    one
Priority:     0
Node:         docker-desktop/192.168.65.3
Start Time:   Thu, 25 Feb 2021 23:32:27 +0000
Labels:       app=nginxdeployment
              pod-template-hash=5cf4dfd5b8
Annotations:  <none>
Status:       Pending
IP:           10.1.3.102
IPs:
  IP:           10.1.3.102
Controlled By:  ReplicaSet/nginxdeployment-5cf4dfd5b8
Containers:
  nginx:
    Container ID:
    Image:          nginx:1.15.666
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gpv6v (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-gpv6v:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gpv6v
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m19s                default-scheduler  Successfully assigned one/nginxdeployment-5cf4dfd5b8-zn7xz to docker-desktop
  Normal   Pulling    35s (x4 over 2m17s)  kubelet            Pulling image "nginx:1.15.666"
  Warning  Failed     31s (x4 over 2m8s)   kubelet            Failed to pull image "nginx:1.15.666": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:1.15.666 not found: manifest unknown: manifest unknown
  Warning  Failed     31s (x4 over 2m8s)   kubelet            Error: ErrImagePull
  Normal   BackOff    9s (x6 over 2m7s)    kubelet            Back-off pulling image "nginx:1.15.666"
  Warning  Failed     9s (x6 over 2m7s)    kubelet            Error: ImagePullBackOff
```
### 5. Woops! Something went crazy wrong here! Rollback the change, so all pods should run nginx:1.15.10 again.
```
$ kubectl rollout undo deployment/nginxdeployment --namespace=one
$ kubectl get deployments --namespace=one

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginxdeployment   15/15   15           15          22m

$ kubectl get pods --namespace=one

NAME                               READY   STATUS    RESTARTS   AGE
nginxdeployment-7cbc44bb97-8r9cp   1/1     Running   0          16m
nginxdeployment-7cbc44bb97-8rnvp   1/1     Running   0          16m
nginxdeployment-7cbc44bb97-f29pk   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-hf4n7   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-k6lz9   1/1     Running   0          16m
nginxdeployment-7cbc44bb97-k8rtc   1/1     Running   0          50s
nginxdeployment-7cbc44bb97-ml9dq   1/1     Running   0          16m
nginxdeployment-7cbc44bb97-mvqlr   1/1     Running   0          50s
nginxdeployment-7cbc44bb97-p26v8   1/1     Running   0          50s
nginxdeployment-7cbc44bb97-rwgtn   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-svw6f   1/1     Running   0          16m
nginxdeployment-7cbc44bb97-tcrg5   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-v9bnh   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-vcqgl   1/1     Running   0          17m
nginxdeployment-7cbc44bb97-vvq5d   1/1     Running   0          16m

$ kubectl get pods --namespace=one -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

CONTAINER   IMAGE
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
nginx       nginx:1.15.10
```

## 5. Secrets and ConfigMaps

### 1. Create a `secret1.yaml`. The yaml should create a secret called `secret1` and store `password:12345678`. Create that secret.
```
$ kubectl create secret generic secret1 --from-literal=password=12345678 --dry-run=client -o yaml > secret1.yaml

apiVersion: v1
data:
  password: MTIzNDU2Nzg=
kind: Secret
metadata:
  creationTimestamp: null
  name: secret1

$ kubectl apply -f secret1.yaml
$ kubectl get secret secret1

NAME      TYPE     DATA   AGE
secret1   Opaque   1      20s
```

### 2. Create a `pod1.yaml` which creates a single pod of image bash . This pod should mount the `secret1` to `/tmp/secret1`. This pod should stay idle after boot. Create that pod.

### 3. Confirm pod1 has access to our password via file-system.

### 4. On your local create a folder drinks and it's content: mkdir drinks; echo ipa > drinks/beer; echo red > drinks/wine; echo sparkling > drinks/water

### 5. Create a ConfigMap containing all files of the folder drinks and their content.

### 6. Make these ConfigMaps available in our pod1 using environment variables.

### 7. Check on pod1 if those environment variables are available.