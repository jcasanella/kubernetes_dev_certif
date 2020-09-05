# Resource Requeriments

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/

## 1. A pod named 'rabbit' is deployed. Identify the CPU requirements set on the Pod

```
kubectl get pods
kubectl describe pod rabbit | grep 'Requests' -A 2
```

## 2. Delete the 'rabbit' Pod.

`kubectl delete pod rabbit`

## 3. Inspect the pod elephant and identify the status.

`kubectl get pod elephant`

## 4. The status 'OOMKilled' indicates that the pod ran out of memory. Identify the memory limit set on the POD.

```
kubectl describe pod elephant | grep 'Requests' -A 5
```

## 5. The elephant runs a process that consume 15Mi of memory. Increase the limit of the elephant pod to 20Mi. Delete and recreate the pod if required. Do not modify anything other than the required fields.

* Pod name: elephant
* ImageName: polinux/stress
* Memory limit: 20Mi

```
kubectl get pod elephant -o yaml > output.yaml
kubectl delete pod elephant
```

Apply the following changes to the pod:

```
resources:
    limits:
        memory: 20Mi
```

Do at the same level as container

`kubectl apply -f output.yaml`