# Service Account

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

## 1. How many Service Accounts exist in the default namespace?

`kubectl get sa`

## 2. What is the secret token used by the default service account?

```
kubectl get secrets
kubectl describe serviceaccount default
```

## 3. We just deployed the Dashboard application. Inspect the deployment. What is the image used by the deployment?

```
kubectl get deployments
kubectl describe deployment web-dashboard |  grep 'Image'
```

## 4. Inspect the Dashboard Application POD and identify the Service Account mounted on it.

```
kubectl get pods
kubectl describe pod web-dashboard-5cb88b85dc-jsjn5 | grep 'serviceaccount'
kubectl describe secret default-token-4jsqv | grep 'service-account.name'
```

## 5. At what location is the ServiceAccount credentials available within the pod?

`kubectl describe pod web-dashboard-5cb88b85dc-jsjn5 | grep 'serviceaccount'`

## 6. The application needs a ServiceAccount with the Right permissions to be created to authenticate to Kubernetes. The 'default' ServiceAccount has limited access. Create a new ServiceAccount named 'dashboard-sa'.

```
kubectl create sa dashboard-sa
kubectl get sa
```

## 7. You shouldn't have to copy and paste the token each time. The Dashboard application is programmed to read token from the secret mount location. However currently, the 'default' service account is mounted. Update the deployment to use the newly created ServiceAccount

* Deployment name: web-dashboard
* Service Account: dashboard-sa
* Deployment Ready

```
kubectl get sa
kubectl edit deployment web-dashboard

and add in the spec section for the container the following entry:

serviceAccount: dashb0ard-sa
```

*Note*: it's a t the same level as containers. A deployment can be edited without to drop the pod.