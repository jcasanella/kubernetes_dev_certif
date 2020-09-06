# Multicontainers

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

## 1. Identify the number of containers running in the 'red' pod.

```
kubectl get pods
kubectl describe pod red | grep -i 'containers' -A 50
```

## 2. Identify the name of the containers running in the 'blue' pod.

```
kubectl get pods
kubectl describe pod blue | grep -i 'containers' -A 50
```

## 3. Create a multi-container pod with 2 containers.

* Name: yellow
* Container 1 Name: lemon
* Container 1 Image: busybox
* Container 2 Name: gold
* Container 2 Image: redis

`kubectl run yellow --image busybox --dry-run=server -o yaml > output.yaml`

edit output.yaml to add the second container and change the name of the first one.

```
  containers:
  - image: busybox
    imagePullPolicy: Always
    name: lemon
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-gbvgh
      readOnly: true
  - image: redis
    name: gold

kubectl apply -f output.yaml
```
## 4. We have deployed an application logging stack in the elastic-stack namespace. Inspect it.

```
kubectl get ns elastic-stack
kubectl describe ns elastic-stack
```

## 5. Inspect the 'app' pod and identify the number of containers in it. It is deployed in the elastic-stack namespace

`kubectl --namespace=elastic-stack describe pod app`

## 6. The 'app'lication outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login. Inspect the log file inside the pod

`kubectl --namespace=elastic-stack exec -it app -- cat /log/app.log | more`

## 7. Edit the pod to add a sidecar container to send logs to ElasticSearch. Mount the log volume to the sidecar container. Only add a new container. Do not modify anything else. Use the spec on the right.

* Name: app
* Container Name: sidecar
* Container Image: kodekloud/filebeat-configured
* Volume Mount: log-volume
* Mount Path: /var/log/event-simulator/
* Existing Container Name: app
* Existing Container Image: kodekloud/event-simulator

```
kubectl --namespace=elastic-stack get pod app -o yaml > output.yaml
kubectl --namespace=elastic-stack delete pod app 

spec:
  containers:
  - image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: app
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /log
      name: log-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-dcbhh
      readOnly: true
  - image: kodekloud/filebeat-configured
    name: sidecar
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

kubectl apply -f output.yaml
```