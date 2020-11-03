# Services and networking

https://kubernetes.io/docs/concepts/services-networking/
https://kubernetes.io/docs/concepts/services-networking/service/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

**ClusterIP** : Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType. 

**NodePort** : Exposes the Service on each Node's IP at a static port (the NodePort ).

## 1. Create a pod with image nginx called nginx and expose its port 80

```
kubectl run nginx --image=nginx --port=80 --expose
kubectl get services
```

Observe a service has been created.

## 2. Confirm that ClusterIP has been created. Also check endpoints

```
kubectl get services
kubectl describe service nginx
kubectl get endpoints
```

## 3. Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

```
kubectl get services | grep ClusterIP
kubectl run busybox --rm --image=busybox -it --restart=Never -- /bin/sh -c 'wget -O- 10.108.53.46:80'
```

## 4. Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

```
kubectl edit service nginx
```

And replace `type: ClusterIP` for `type: NodePort`

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2020-10-26T22:36:25Z"
  name: nginx
  namespace: default
  resourceVersion: "228285"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: a14d57d5-7a61-44df-96fd-99f06bfcf159
spec:
  clusterIP: 10.108.53.46
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

```
kubbectl describe service nginx
kubectl delete service nginx
kubectl delete pod nginx
kubectl get svc
kubectl get pods
```

## 5. Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo
  labels:
    app: foo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: foo
  template:
    metadata:
      labels:
        app: foo
    spec:
      containers:
      - name: nginx
        image: dgkatnasios/simpleapp
        ports:
        - containerPort: 8080
```

```
kubectl apply -f output.yaml
kubectl get deployments
```

other option

```
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
```

## 6. Get the pod IPs. Create a temp busybox pod and trying hitting them on port 8080

```
kubectl get pods -o wide
```
```
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh 
wget -O- 10.1.0.77:8080
```

## 7. Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

```
kubectl create service clusterip foo --tcp 6262:8080
kubectl get service foo 
kubectl get endpoints foo
```

Other option is:

```
kubectl expose deploy foo --port=6262 --target-port=8080
```

## 8. Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

```
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh 
wget -O- foo:6262
```

```
kubectl delete svc foo
kubectl get svc
kubectl delete deployment foo
kubectl get deployments
```

## 9. Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it