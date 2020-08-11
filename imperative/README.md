# Imperative exercises

## Deploy a pod named nginx-pod using the nginx:alpine image.

`kubectl run nginx-pod --image=nginx:alpine`

## Deploy a redis pod using the redis:alpine image with the labels set to tier=db.Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.

```
kubectl run redis --image redis:alpine -o yaml --dry-run=client > output.yaml

edit output.yaml to add the label

apiVersion: v1
kind: Pod
metadata:
    creationTimestamp: null
	labels:
	    run: redis
		tier: db
	name: redis
spec:
    containers:
	- image: redis: alpine
	  name: redis
	  resources: {}
	dnsPolicy: ClusterFirst
	restartPolicy: Always
status: {}

kubectl apply -f output.yaml
```

## Create a service redis-service to expose the redis application within the cluster on port 6379. Use imperative commands

```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml > output2.yaml
just to test is using the correct label
kubectl apply -f output2.yaml
```

## Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas. Try to use imperative commands only. Do not create definition files.

```
kubectl create deployment webapp --image=kodekloud/webapp-color
kubectl scale deployments/webapp --replicas=3
```

## Create a new pod called custom-nginx using the nginx image and expose it on container port 8080

`kubectl run custom-nginx --image=nginx --port=8080`

## Create a new namespace called dev-ns.

`kubectl create namespace dev-ns`

## Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.

```
kubectl --namespace=dev-ns create deployment redis-deploy --image=redis
kubectl --namespace=dev-ns scale deployments/redis-deploy --replicas=2
```

## Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.

```
kubectl run httpd --image httpd:alpine
kubectl expose pod httpd --port=80 --target-port=80 --type=ClusterIP --name httpd 
```
