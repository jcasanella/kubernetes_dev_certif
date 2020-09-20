# Ingres Networking I

https://kubernetes.io/docs/concepts/services-networking/ingress/
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

## 1. We have deployed Ingress Controller, resources and applications. Explore the setup. Note: They are in different namespaces.

```
kubectl get deployments --all-namespaces
```

## 2. Which namespace is the Ingress Resource deployed in?

```
kubectl get ingress --all-namespaces
```

## 3. What is the Host configured on the ingress-resource? The host entry defines the domain name that users use to reach the application like www.google.com

```
kubectl describe ingress --namespace app-space
```

## 4. If the requirement does not match any of the configured paths what service are the requests forwarded to?

After running the previous command, check the value of `default-backend`

## 5. You are requested to change the URLs at which the applications are made available.

* Ingress: ingress-wear-watch
* Path: /stream
* Backend Service: video-service
* Backend Service Port: 8080

```
kubectl get ingress ingress-wear-watch --namespace app-space -o yaml > output.yaml
```

Replace `path: /watch` for `path: /stream`

```
kubectl delete ingress ingress-wear-watch --namespace app-space
kubectl apply -f output.yaml
```

## 6. Due to increased demand, your business decides to take on a new venture. You acquired a food delivery company. Their applications have been migrated over to your cluster. Inspect the new deployments in the app-space.

```
kubectl get deployments,svc --namespace app-space
```

Using the svc we check either the Service

## 7. You are requested to add a new path to your ingress to make the food delivery application available to your customers. Make the new application available at /eat.

* Ingress: ingress-wear-watch
* Path: /eat
* Backend Service: food-service
* Backend Service Port: 8080

```
kubectl get ingress ingress-wear-watch --namespace app-space -o yaml > output.yaml
```

and another new backend: 

```
      - backend:
          serviceName: food-service
          servicePort: 8080
        path: /eat
```

and delete the existing ingress:

```
kubectl delete ingress ingress-wear-watch --namespace app-space
kubectl apply -f output.yaml
```

## 8. A new payment service has been introduced. Since it is critical, the new application is deployed in its own namespace. Identify the namespace in which the new application is deployed

```
kubectl get deployments --all-namespaces
kubectl get deployments,svc --namespace critical-space
```

## 9. You are requested to make the new application available at /pay. Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.

* Ingress Created
* Path: /pay
* Configure correct backend service
* Configure correct backend port

Edit the file created in  exercise 7.
Change namespace to  `critical-space`. Also change the name of the application to `name: ingress-pay`. Remove all the backends, just keep one backend and edit this one.

```
      paths:
      - backend:
          serviceName: pay-service
          servicePort: 8282
        path: /pay
        pathType: ImplementationSpecific
```

And create the new ingress with `kubectl apply -f output.yaml`

```
kubectl describe ingress ingress-pay --namespace critical-space
```
