# Liveness and readiness probes

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

## 1. Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Edit the file and add the `livenessProbe` section

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
    livenessProbe:
      exec:
        command:
        - ls
      initialDelaySeconds: 5
      periodSeconds: 5
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl apply -f .\output.yaml
kubectl describe pod nginx | grep -i liveness
```

Look for this section in the describe output: `Liveness:`

Delete the pod: 

```
kubectl delete -f .\output.yaml
```

## 2. Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

Done in the previous exercise. Important settings:

**initialDelaySeconds**: wait until n seconds before starting the probe

**periodSeconds**: seconds to try it its live

## 3. Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > output.yaml
```

Add the section `readinessProbe`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
kubectl apply -f output.yaml
kubectl describe pod nginx
kubectl delete -f output.yaml
```

