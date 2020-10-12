# Deployments

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

## 1. Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

```
kubectl create deployment nginx --image=nginx:1.7.8 --replicas=2 --port=80
kubectl get deployments nginx | grep nginx  
```

## 2. View the YAML of this deployment

```
kubectl get deployment nginx -o yaml > output.yaml
```

## 3. View the YAML of the replica set that was created by this deployment

```
kubectl get replicasets
kubectl get rs nginx-8f5cc57cf -o yaml 
```

OR you can find rs directly by:

```
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
```

## 4. Get the YAML for one of the pods

```
kubectl get pods -l app=nginx   
kubectl get pods nginx-8f5cc57cf-6j5h2 -o yaml  
```

## 5. Check how the deployment rollout is going

```
kubectl rollout status deployment/nginx
```

other option

```
kubectl rollout status deploy nginx
```

## 6. Update the nginx image to nginx:1.7.9

```
kubectl edit deployment nginx
kubectl rollout status deploy nginx
```

## 7. Check the rollout history and confirm that the replicas are OK

```
kubectl rollout history deployment nginx
kubectl get rs -l app=nginx
```

## 8. Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8)

```
kubectl rollout undo deployment nginx
kubectl get pods -l app=nginx
kubectl describe pod nginx-5d59d67564-6blfm
```

## 9. Do an on purpose update of the deployment with a wrong image nginx:1.91

```
kubectl edit deployment nginx
```

## 10. Verify that something's wrong with the rollout

```
kubectl rollout status deploy nginx
kubectl get pods -l app=nginx
```

## 11. Return the deployment to the second revision (number 2) and verify the image is nginx:1.7.9