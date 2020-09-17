# Services

https://kubernetes.io/docs/concepts/services-networking/service/

## 1. How many Services exist on the system? In the current default namespace.

```
kubectl get services
```

or

```
kubectl get svc
```

## 2. What is the type of the default 'kubernetes' service?

```
kubectl get service kubernetes
```

## 3. What is the 'targetPort' configured on the 'kubernetes' service?

```
kubectl describe service kubernetes
```

## 4. How many labels are configured on the 'kubernetes' service?

```
kubectl describe service kubernetes
```

## 5. How many Endpoints are attached on the 'kubernetes' service?

```
kubectl describe service kubernetes
```

## 6. How many Deployments exist on the system now? in the current(default) namespace

```
kubectl get deployments
```

## 7. What is the image used to create the pods in the deployment?

```
kubectl describe deployment simple-webapp-deployment | grep -i 'image'
```

## 8. Create a new service to access the web application using the service-definition-1.yaml file


* Name: webapp-service
* Type: NodePort
* targetPort: 8080
* port: 8080
* nodePort: 30080; 
* selector: simple-webapp

```
kubectl expose deployment simple-webapp-deployment --name=webapp-service --target-port=8080 --type=NodePort --port=8080 --dry-run=client -o yaml > svc.yaml
```

Edit the file and add nodePort: 30080

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: webapp-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    nodePort: 30080
  selector:
    name: simple-webapp
  type: NodePort
status:
  loadBalancer: {}
```

and run `kubectl apply -f svc.yaml`