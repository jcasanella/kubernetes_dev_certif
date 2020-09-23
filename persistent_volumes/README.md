# Persistent Volumes

https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

## 1. We have deployed a POD. Inspect the POD and wait for it to start running.

```
kubectl get pods
kubectl describe pod webapp
```

## 2. The application stores logs at location /log/app.log. View the logs.

```
kubectl exec -it webapp -- cat /log/app.log
```

If the POD was to get deleted now, we're not able to view these logs.

## 3. Configure a volume to store these logs at /var/log/webapp on the host. Use the spec given

* Name: webapp
* Image Name: kodekloud/event-simulator
* Volume HostPath: /var/log/webapp
* Volume Mount: /log

```
kubectl delete pod webapp
```

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: log
    hostPath:
      path: /var/log/webapp
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    imagePullPolicy: Always
    name: event-simulator
    image: kodekloud/event-simulator
    volumeMounts:
    - name: log
      mountPath: /log
```

```
kubectl apply -f pod.yaml
```

## 4. Create a 'Persistent Volume' with the given specification.

* Volume Name: pv-log
* Storage: 100Mi
* Access modes: ReadWriteMany
* Host Path: /pv/log

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/log
```
```
kubectl explain pv --recursive
kubectl create -f pv.yaml
```

## 5. Let us claim some of that storage for our application. Create a 'Persistent Volume Claim' with the given specification.

* Volume Name: claim-log-1
* Storage Request: 50Mi
* Access modes: ReadWriteOnce

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

```
kubectl apply -f pvc.yaml
```

## 6. What is the state of the Persistent Volume Claim?

`kubectl get pvc`

## 7. What is the state of the Persistent Volume?

`kubectl get pv`

## 8. Update the Access Mode on the claim to bind it to the PVC. Delete and recreate the claim-log-1

* Volume Name: claim-log-1
* Storage Request: 50Mi
* PVol: pv-log
* Status: Bound

**Note**: To bind the PersistentVolume and the PersistentVolumeClaim, both of them must be in the same accessModes.

```
kubectl delete pvc claim-log-1
```

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```

```
kubectl apply -f pvc.yaml
```
## 9. Update the webapp pod to use the persistent volume claim as its storage. Replace hostPath configured earlier with the newly created PersistentVolumeClaim

* Name: webapp
* Image Name: kodekloud/event-simulator
* Volume: PersistentVolumeClaim=claim-log-1
* Volume Mount: /log

```
kubectl get pod webapp -o yaml > pod.yaml
kubectl delete pod webapp
```

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    volumeMounts:
    - mountPath: /log
      name: log
  restartPolicy: Never
  volumes:
    - name: log
      persistentVolumeClaim:
        claimName: claim-log-1
```

```
kubectl apply -f pod2.yaml
```

## 10. Try deleting the PVC and notice what happens.

```
kubectl get pvc
kubectl delete pvc claim-log-1
```

the PVC stuck in 'Terminating' state until we drop the POD