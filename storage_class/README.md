# Storage Class

https://kubernetes.io/docs/concepts/storage/storage-classes/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims

## 1. How many StorageClasses exist in the cluster right now?

```
kubectl get sc --no-headers=true --all-namespaces
```

## 2. What is the Provisioner used for the storage class called portworx-io-priority-high?

```
kubectl describe sc portworx-io-priority-high
```

## 3. Is there a PersistentVolumeClaim that is consuming the PersistentVolume called local-pv?

```
kubectl get pvc --all-namespaces
```

## 4. Let's fix that. Create a new PersistentVolumeClaim by the name of local-pvc that should bind to the volume local-pv Inspect the pv local-pv for the specs.

* PVC: local-pvc
* Correct Access Mode?
* Correct StorageClass Used?
* PVC requests volume size = 500Mi?

```
kubectl describe pv local-pv
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```

```
kubectl apply -f pvc.yaml
```

## 5. What is the status of the newly created Persistent Volume Claim?

```
kubectl get pvc local-pvc
```

## 6. Create a new pod called nginx with the image nginx:alpine. The Pod should make use of the PVC local-pvc and mount the volume at the path /var/www/html. The PV local-pv should in a bound state.

* Pod created with the correct Image?
* Pod uses PVC called local-pvc?
* local-pv bound?
* nginx pod running?
* Volume mounted at the correct path?

```
kubectl run nginx --image nginx --dry-run client -o yaml > output.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
    - mountPath: "/var/www/html"
      name: local-persistent-storage
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

```
kubectl apply -f pod.yaml
```

## 7. Create a new Storage Class called delayed-volume-sc that makes use of the below specs:

* provisioner: kubernetes.io/no-provisioner
* volumeBindingMode: WaitForFirstConsumer

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer





