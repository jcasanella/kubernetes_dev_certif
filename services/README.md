# Services

https://kubernetes.io/docs/concepts/services-networking/service/

A pod is ephemeral. Each pod is assigned a unique IP address. If a pod that belongs to a replication controller dies, then it is recreated and may be given a different IP address. Further, additional pods may be created using replication controllers. This makes it difficult for an application server to access a database such as Couchbase using its IP address. A service is an abstraction that defines a logical set of pods and a policy by which to access them. The IP address assigned to a service does not change over time, and thus can be relied upon by other pods. Typically, the pods belonging to a service are defined by a label selector. This is similar to how pods belong to a replication controller

We use `kubectl expose` to create a service for existing pods. Services allows us to connect into these pods. Once a service is created, CoreDNS will allow us to resolve by name. There're different types of services:anybody can connect to it. Our code must be changed to connect to that new port.
* **LoadBalancer**: an external load balancer is allocated for the service. Like AWS, Azure, GCE, OpenStack...
* **ExternalName**: The DNS entry managed by CoreDNS will just be a `CNAME` to a provided record. No port, no IP address, no nothing else is allocated.

Under the hood is using kube-proxy.

Remember CoreDNS is part of the control pane.

## 1. How many Services exist on the system? In the current default namespace.

```
kubectl get services

* **ClusterIP**: It's the default.  A virtual IP address is allocated for the service, this IP address is reachable only from within the cluster. Our code can connect to the Pod using the original port number.
* **NodePort**: A port is allocated for the service (by default, in the range 30000-32768) That port is made available on all your nodes and 
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