# State Persistence

https://kubernetes.io/docs/concepts/storage/volumes/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

## 1. Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

```
kubectl run busybox --image busybox -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > output.yaml 
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: 
    - name: myvolume 
      mountPath: /etc/foo 
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 
    volumeMounts: 
    - name: myvolume 
      mountPath: /etc/foo 
  volumes: 
  - name: myvolume 
    emptyDir: {} 
```
```
kubectl create -f output.yaml
```
```
kubectl exec -it busybox2 -- /bin/sh
cat /etc/passwd | awk -F ':' '{print $1}' > /etc/foo/passwd
cat /etc/foo/passwd
```

```
kubectl exec -it busybox -- /bin/sh
cat /etc/foo/passwd
```

## 2. Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myvolume
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  storageClassName: normal
  hostPath:
    path: /etc/foo
```

```
kubectl apply -f output.yaml
kubectl get pv
```

## 3. Create a PersistentVolumeClaim for this storage class, called mypvc, a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: normal
```

```
kubectl apply -f output.yaml
kubectl get pvc
kubectl get pv
```

## 4. Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

```
kubectl run busybox --image=busybox -o yaml --dry-run=client > output.yaml -- /bin/sh -c 'sleep 3600'
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: mypvc
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
    volumeMounts:
      - mountPath: "/etc/foo"
        name: task-pv-storage
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl create -f output.yaml
kubectl exec -it busybox -- /bin/sh
cp /etc/passwd /etc/foo/.
```

## 5. Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?