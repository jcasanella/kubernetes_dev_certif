# Readiness and Liveness

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

## 1. We have deployed a simple web application. Inspect the PODs and Services

```
kubectl get pods
kubectl get pod simple-webapp-1 -o wide
kubectl describe pod simple-webapp-1
kubectl get services
kubectl get service kubernetes
kubectl get service webapp-service
```

## 2. Update the newly created pod 'simple-webapp-2' with a readinessProbe using the given spec

* Pod Name: simple-webapp-2
* Image Name: kodekloud/webapp-delayed-start
* Readiness Probe: httpGet
* Http Probe: /ready
* Http Port: 8080

kubectl get pod simple-webapp-2 -o yaml > output.yaml

spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080

kubectl delete pod simple-webapp-2
kubectl apply -f output.yaml

## 3. Update both the pods with a livenessProbe using the given spec

* Pod Name: simple-webapp-1
* Image Name: kodekloud/webapp-delayed-start
* Liveness Probe: httpGet
* Http Probe: /live
* Http Port: 8080
* Period Seconds: 1
* Initial Delay: 80
* Pod Name: simple-webapp-2
* Image Name: kodekloud/webapp-delayed-start
* Liveness Probe: httpGet
* Http Probe: /live
* Http Port: 8080
* Initial Delay: 80
* Period Seconds: 1

```
kubectl get pods
kubectl get pod simple-webapp-1 -o yaml > output1.yaml
kubectl get pod simple-webapp-2 -o yaml > output2.yaml

spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 1
      initialDelaySeconds: 80

kubectl delete pod simple-webapp-1
kubectl delete pod simple-webapp-2
kubectl apply -f output1.yaml
kubectl apply -f output2.yaml
```


