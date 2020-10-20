# Configuration

https://kubernetes.io/docs/concepts/configuration/configmap/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

## ConfigMaps

### 1. Create a configmap named config with values foo=lala,foo2=lolo

```
kubectl create configmap my-config --from-literal=foo=lala --from-literal=foo2=lolo
kubectl get configmaps
kubectl describe cm my-config
```

### 2. Display its values

```
kubectl describe cm my-config
```

### 3. Create and display a configmap from a file

```
echo "foo=lala\nfoo2=lolo" > cm.txt
kubectl create configmap my-config2 --from-file=cm.txt
kubectl get cm
kubectl describe cm my-config2
```

### 4. Create and display a configmap from a .env file

```
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
kubectl create configmap my-config3 --from-env-file=config.env
kubectl get cm
kubectl describe cm my-config3
```

### 5. Create and display a configmap from a file, giving the key 'special'

```
echo -e "var3=val3\nvar4=val4" > config4.txt
kubectl create configmap my-config4 --from-file=special=config4.txt
kubectl describe cm my-config4
```

### 6. Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

```
kubectl create cm options --from-literal=var5=val5
```

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
    # Define the environment variable
    - name: option
      valueFrom:
        configMapKeyRef:
          # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
          name: options
          # Specify the key associated with the value
          key: var5
```

```
kubectl create -f output.yaml
kubectl exec -it nginx -- env | grep option
```

### 7. Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

```
kubectl create cm anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

edit outputfile and add the section `envFrom`

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
    envFrom:
    - configMapRef:
        name: anotherone
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

kubectl create -f output.yaml
kubectl exec -it nginx -- env

### 8. Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

```
kubectl create cm cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Add the `volumes` section and `volumeMounts`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  restartPolicy: Never
```
```
kubectl apply -f output.yaml
kubectl describe pod nginx
kubectl logs nginx
kubectl exec -it nginx -- ls /etc/lala
```

## Security Context

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

### 1. Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Edit the output to add `securityContext:`

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    runAsUser: 101
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 2. Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added on its single container

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Edit the output.yaml to add the `securityContext:` section

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    capabilities:
      add: ["NET_ADMIN", "SYS_TIME"]
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Requests and Limits

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/

### 1. Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

```
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: 512Mi
        cpu: 200m
      requests:
        memory: 256Mi
        cpu: 100m
```