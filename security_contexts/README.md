# Security Contexts

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

## 1. Edit pod ubuntu-sleeper to run the sleep process with user ID 1010
 * Pod name ubuntu-sleeper
 * image name ubuntu
 * SecurityContext: User 1010

```
kubectl delete ubuntu-sleeper

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu

kubectl apply -f output.yaml
```
## 2. Try to run the below command in the pod 'ubuntu-sleeper' to set the date. Are you allowed to set date on the POD?

* date -s '19 APR 2012 11:14:00'

`kubectl exec -it ubuntu-sleeper --  date -s '19 APR 2012 11:14:00'`

Output: `No permissions to run`

## 3. Update pod 'ubuntu-sleeper' to run as Root user and with the 'SYS_TIME' capability. Note: Only make the necessary changes. Do not modify the name of the pod.

* Pod Name: ubuntu-sleeper
* Image Name: ubuntu
* SecurityContext: Capability SYS_TIME

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu
    securityContext:
       capabilities:
          add: ["SYS_TIME"]
```


## 4. Now try to run the below command in the pod to set the date. If the security capability was added correctly, it should work. If it doesn't make sure you changed the user back to root.

* date -s '19 APR 2012 11:14:00'

`kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'`


