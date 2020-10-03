# Multi Containers Pods

https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## Some theory

**Init containers** can contain utilities or setup scripts not present in an app image.  A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

* Init containers are exactly like regular containers, except:
* Init containers always run to completion. Each init container must complete successfully before the next one starts.

If you specify multiple init containers for a Pod, Kubelet runs each init container sequentially. Each init container must succeed before the next can run. When all of the init containers have run to completion, Kubelet initializes the application containers for the Pod and runs them as usual.


## 1. Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

```
kubectl run busybox --image=busybox --dry-run=client -o yaml > output.yaml       
```

Edit output.yaml to create 2 containers:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
    resources: {}
  - image: busybox
    name: busybox2
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
cat << EOF > output.yaml
```

Remeber to add EOF at the end of the file

```
kubectl apply -f output.yaml
kubectl exec -it busybox -c busybox2 -- /bin/sh  
ls
```

## 2. Create nginx pod exposed at port 80. Add an busybox `init container` which downloads the k8s page by `"wget -O /work-dir/index.html http://kubernetes.io"`. Make a volume of type `emptyDir` and mount it in both pods. For nginx mount it on `"/usr/share/nginx/html"` and for the initcontainer use mount it on `"/work-dir"`. When done, get the IP of the nginx pod and create a busybox pod and run wget -O- IP

```
kubectl run nginx --image=nginx --port 80 --dry-run=client -o yaml > output.yaml
```

Edit output.yaml and add the container busybox with the command:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: box
  name: box
spec:
  initContainers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'wget work-dir/index.html http://kubernetes.io']  
    volumeMounts:
    - mountPath: /work-dir
      name: cache-volume
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

```
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box --image=busybox --restart=Never -ti --rm -- /bin/sh -c "wget -O- IP"

# you can do some cleanup
kubectl delete po box
```



