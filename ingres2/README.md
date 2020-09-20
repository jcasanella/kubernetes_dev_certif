# Ingres2 - Installing an ingress 

## 1. We have deployed two applications. Explore the setup. Note: They are in different namespaces.

```
kubectl get deployments,svc --all-namespaces
```

## 2. Let us now deploy an Ingress Controller. First, create a namespace called 'ingress-space' We will isolate all ingress related objects into its own namespace.

* Name: ingress-space

```
kubectl create namespace ingress-space
```

## 3. The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object in the ingress-space. 

* Name: nginx-configuration

```
kubectl create configmap nginx-configuration --namespace ingress-space
kubectl get configmaps --namespace ingress-space
```

## 4. The NGINX Ingress Controller requires a ServiceAccount. Create a ServiceAccount in the ingress-space.

* Name: ingress-serviceaccount

```
kubectl create sa ingress-serviceaccount --namespace ingress-space
kubectl get serviceaccounts --namespace ingress-space
```

## 5. We have created the Roles and RoleBindings for the ServiceAccount. Check it out!

```
kubectl get roles,rolebindings --namespace ingress-space
```

## 6. Let us now deploy the Ingress Controller. Create a deployment using the file given. The Deployment configuration is given at /root/ingress-controller.yaml. There are several issues with it. Try to fix them.

* Deployed in the correct namespace.
* Replicas: 1
* Use the right image

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
                containerPort: 80
            - name: https
              containerPort: 443
```

Errors:

* namespace: ingress-space
* apiVersion: apps/v1
* drop line ---
* align: containerPort: 80

Also u can do with the following command

```
kubectl expose deployment ingress-controller -n ingress-space --name ingress --port 80 --target-port 80 --type NodePort --dry-run=client -o yaml > ingress-svc.yaml
```

add the `namespace: ingress-space` and `nodePort: 30080`

```
kubectl apply -f ingress-controller.yaml
kubectl describe deployment ingress-controller --namespace ingress-space
```

## 7. Let us now create a service to make Ingress available to external users. Create a service following the given specs.

* Name: ingress
* Type: NodePort
* Port: 80
* TargetPort: 80
* NodePort: 30080
* Use the right selector

```
kubectl expose deployment -n ingress-space ingress-controller --type=NodePort --port=80 --name=ingress --dry-run=client -o yaml >ingress.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  selector:
    app: ingres
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Add to the previous yaml the namespace and the nodePort

```
kubectl apply -f ingress.yaml
```

## 8. Create the ingress resource to make the applications available at /wear and /watch on the Ingress service. Create the ingress in the app-space

* Ingress Created
* Path: /wear
* Path: /watch
* Configure correct backend service for /wear
* Configure correct backend service for /watch
* Configure correct backend port for /wear service
* Configure correct backend port for /watch service

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080
```

`kubectl apply -f output.yaml`