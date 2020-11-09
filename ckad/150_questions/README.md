# Practice enough with these questions  for the CKAD exam

https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552

## Core Concepts (13%)

https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-
https://kubernetes.io/docs/concepts/workloads/pods/
https://kubernetes.io/docs/reference/kubectl/jsonpath/

### 1. List all the namespaces in the cluster
```
kubectl get ns
```

### 2. List all the pods in all namespaces
```
kubectl get pods --all-namespaces
```

### 3. List all the pods in the particular namespace
```
kubectl get pods -n kube-system
```

### 4. List all the services in the particular namespace
```
kubectl get svc -n kube-system
```

### 5. List all the pods showing name and namespace with a json path expression
```
kubectl explain pods.metadata
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"
```

### 6. Create an nginx pod in a default namespace and verify the pod running
```
kubectl run nginx --image=nginx --restart=Never
kubectl get pods -l 'run=nginx'
```

### 7. Create the same nginx pod with a yaml file
```
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > output.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```
kubectl create -f output.yaml
kubectl get pods -l 'run=nginx'
```

### 8. Output the yaml file of the pod you just created
```
kubectl get pod nginx -o yaml
```

### 9. Get the complete details of the pod you just created
```
kubectl describe pod nginx
```

### 10. Delete the pod you just created
```
kubectl delete -f output.yaml
```

or

```
kubectl delete pod nginx
```

### 11. Delete the pod you just created without any delay (force delete)
```
kubectl delete pod nginx --force
```

### 12. Create the nginx pod with version 1.17.4 and expose it on port 80
```
kubectl run nginx --image=nginx:1.17.4 --port=80
```

### 13. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated
```
kubectl edit pod nginx
kubectl describe pod nginx
```

### 14. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes
```
kubectl edit pod nginx
kubectl describe pod nginx
kubectl get pod nginx -w
```

### 15. Check the Image version without the describe command
```
kubectl explain pods.spec.containers
kubectl get pod nginx -o jsonpath="{.spec.containers[].image}" | awk -F ":" '{ print $2 }'
```

### 16. Create the nginx pod and execute the simple shell on the pod
```
kubectl run nginx --image=nginx --restart=Never
kubectl exec -it nginx -- /bin/sh
```

### 17. Get the IP Address of the pod you just created
```
get pod nginx  -o wide --no-headers
```

### 18. Create a busybox pod and run command ls while creating it and check the logs
```
kubectl run busybox --image=busybox -- ls
kubectl logs busybox
```

### 19. If pod crashed check the previous logs of the pod
```
kubectl logs --help
kubectl logs busybox -p
```

### 20. Create a busybox pod with command sleep 3600
```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'sleep 3600'
```

### 21. Check the connection of the nginx pod from the busybox pod
```
kubectl get pods -o wide
kubectl exec -it busybox -- /bin/sh
wget -O- 10.1.0.108:80
```

### 22. Create a busybox pod and echo message ‘How are you’ and delete it manually
```
kubectl run busybox --image=nginx --restart=Never -it -- echo "How are you"
kubectl delete pod busybox
```

### 23. Create a busybox pod and echo message ‘How are you’ and have it deleted immediately
```
kubectl run busybox --image=nginx --restart=Never --rm -it -- echo "How are you"
```

### 24. Create an nginx pod and list the pod with different levels of verbosity
```
kubectl run nginx --image=nginx --restart=Never --port=80
kubectl get po nginx --v=7
kubectl get po nginx --v=8
kubectl get po nginx --v=9
``` 

### 25. List the nginx pod with custom columns POD_NAME and POD_STATUS
```
kubectl get pods --help # look for custom columns
kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
```

### 26. List all the pods sorted by name

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

```
kubectl explain pod.metadata
kubectl get pods --sort-by=.metadata.name
```

### 27. List all the pods sorted by created timestamp
```
kubectl explain pod.metadata
kubectl get pods --sort-by=.metadata.creationTimestamp
```

## Multi-Container Pods (10%)

### 28. Create a Pod with three busy box containers with commands “ls; sleep 3600;”, “echo Hello World; sleep 3600;” and “echo this is the third container; sleep 3600” respectively and check the status
```
kubectl run busybox --image=busybox --dry-run=client -o yaml > output.yaml -- /bin/sh -c "ls; sleep 3600"
```
and add the other 2 containers to the pod definition.
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - ls; sleep 3600
    image: busybox
    name: busybox1
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo 'Hello World'; sleep 3600
    image: busybox
    name: busybox2
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo 'This is the third container'; sleep 3600
    image: busybox
    name: busybox3
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```
kubectl get pods -l run=busybox
```

### 29. Check the logs of each container that you just created
```
kubectl logs busybox --help
kubectl logs busybox -c busybox1
kubectl logs busybox -c busybox2
kubectl logs busybox -c busybox3
```

### 30. Check the previous logs of the second container busybox2 if any
```
kubectl logs busybox -c busybox2 -p
```

### 31. Run command ls in the third container busybox3 of the above pod
```
kubectl exec -it busybox -c busybox3 -- /bin/sh -c 'ls'
```

### 32. How metrics of the above pod containers and puts them into the file.log and verify

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top

```
kubectl top pod --help
kubectl top pod busybox --containers
kubectl top pod busybox --containers > file.log
cat file.log
```

### 33. Create a Pod with main container busybox and which executes this “while true; do echo ‘Hi I am from Main container’ >> /var/log/index.html; sleep 5; done” and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running.

https://www.magalix.com/blog/the-sidecar-pattern
https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d
https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-init-container-pattern-7a757742de6b
https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-adaptor-container-pattern-97674285983c
https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a
https://kubernetes.io/docs/concepts/storage/volumes/

```
kubectl run busybox --image=busybox --dry-run=client -o yaml > output.yaml -- /bin/sh -c "while true; do echo 'Hi I am from Main container' >> /var/log/index.html; sleep 5; done"

kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > nginx.yaml
```

add nginx container and expose port 80, we can use the template created in the previouscmd

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do echo Hi I am from Main container >> /var/log/index.html; sleep
      5; done
    image: busybox
    name: busybox
    resources: {}
    volumeMounts:
    - mountPath: /var/log
      name: cache-volume
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl apply -f output.yaml

kubectl exec -it  busybox -c busybox -- /bin/sh -c 'cat /var/log/index.html'
kubectl exec -it busybox -c nginx -- /bin/sh -c 'cat /usr/share/nginx/html'
```

## Pod Design (20%)

 ### 34. Get the pods with label information
```
kubectl get pods --show-labels
```
### 35. Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev

```
#!/bin/bash

for i in {1..5}
do
  echo "Creating pod nginx${i}"
  if [[ $i -lt 3 ]]
  then
    kubectl run nginx${i} --image=nginx --labels="env=prod"
  else
    kubectl run nginx${i} --image=nginx --labels="env=prod"
  fi
done
```

```
kubectl get pods -l 'env=prod'
kubectl get pods -l 'env=dev'
```

### 36. Verify all the pods are created with correct labels
```
kubectl get pods --show-labels
```

### 37. Get the pods with label env=dev and also output the labels
```
kubectl get pods -l 'env=dev' --show-labels
```

### 38. Get the pods with label env=prod and also output the labels
```
kubectl get pods -l 'env=prod' --show-labels
```

### 39. Get the pods with label env
```
kubectl get pods
kubectl get pods -l env
```

### 40. Get the pods with labels env=dev and env=prod
```
kubectl get pods -l 'env in (dev,prod)'
```

### 41. Get the pods with labels env=dev and env=prod and output the labels as well
```
kubectl get pods -l 'env in (dev,prod)' --show-labels
```

### 42. Change the label for one of the pod to env=uat and list all the pods to verify

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#label

```
kubectl edit pod nginx5
```
Change the label
```
kubectl get pods --show-labels
```

Other option is:
```
kubectl label pod/nginx-dev3 env=uat --overwrite
```

### 43. Remove the labels for the pods that we created now and verify all the labels are removed

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

This command in windows, it fails, but should work in linux:
```
kubectl label pod nginx{1..2} env-
```

```
kubectl label pod nginx1 env-
kubectl label pod nginx2 env-
kubectl label pod nginx3 env-
kubectl label pod nginx4 env-
kubectl label pod nginx5 env-

kubectl get pods --show-labels | grep nginx
```

### 44. Let’s add the label app=nginx for all the pods and verify

In windows does not work, to test in linux
```
kubectl label pod nginx-dev{1..3} app=nginx
kubectl label pod nginx-prod{1..2} app=nginx
kubectl get po --show-labels
```

```
#!/bin/bash

for i in {1..5}
do
  kubectl label pod nginx${i} app=nginx
done
```

```
kubectl get pods -l 'app=nginx'
```

### 45. Get all the nodes with labels (if using minikube you would get only master node)
```
kubectl get nodes --show-labels
```

### 46. Label the node (minikube if you are using) nodeName=nginxnode
```
kubectl get pods -o wide | awk {'print $7'}
kubectl label node docker-desktop nodeName=nginxnode
kubectl get nodes -l nodeName=nginxnode --show-labels
```

### 47. Create a Pod that will be deployed on this node with the label nodeName=nginxnode

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

```
kubectl run nginx --image=nginx -o yaml --dry-run=client > output.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Add the node selector:
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  nodeSelector:
    nodeName: nginxnode
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl describe po nginx | grep Node-Selectors
```

### 48. Annotate the pods with name=webapp

```
#!/bin/bash

for i in {1..5}
do
  kubectl annotate pods nginx${i} name=webapp
done
```

```
kubectl describe pod nginx1 | grep Annotations
```

Other option:

```
kubectl annotate pod nginx-dev{1..3} name=webapp
kubectl annotate pod nginx-prod{1..2} name=webapp
```

### 49. Remove the annotations on the pods and verify
```
#!/bin/bash

for i in {1..5}
do
  kubectl annotate pods nginx${i} name-
done
```

Other option:

```
kubectl annotate pod nginx-dev{1..3} name-
kubectl annotate pod nginx-prod{1..2} name-
```

```
kubectl describe pod nginx{1..5} | grep -i annotations
```

### 50. Remove all the pods that we created so far
```
kubectl delete pods --help
kubectl delete pods --all
kubectl get pods
```

### 51. Create a deployment called webapp with image nginx with 5 replicas
```
kubectl create deployment webapp --image=nginx --replicas=5
kubectl get deployments
```

### 52. Output the yaml file of the deployment you just created
```
kubectl get deploy webapp -o yaml
```

### 53. Get the pods of this deployment
```
kubectl get deployments --show-labels
kubectl get pods -l app=webapp
```

### 54. Scale the deployment from 5 replicas to 20 replicas and verify
```
kubectl edit deployment webapp
```
Change replicas to 20
```
kubectl get deployments --show-labels
kubectl get pods -l app=webapp --no-headers | wc -l
```

### 55. Get the deployment rollout status

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout
```
kubectl rollout status deployment webapp
```

### 56. Get the replicaset that created with this deployment
```
kubectl get deployments --show-labels
kubectl get replicaset -l app=webapp
```

### 57. Get the yaml of the replicaset and pods of this deployment
```
kubectl get rs,pods -l app=webapp -o yaml
```

other option is

```
kubectl get rs -l app=webapp -o yaml
kubectl get po -l app=webapp -o yaml
```

### 58. Delete the deployment you just created and watch all the pods are also being deleted
```
kubectl delete deployment webapp
kubectl get pods -l app=webapp
```

### 59. Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version
```
kubectl create deployment webapp --image=nginx:1.17.1 --port=80
kubectl describe pod webapp-5b94956f7-pwznz | grep -i image
```

### 60. Update the deployment with the image version 1.17.4 and verify
```
kubectl edit webapp
kubectl get deployments --show-labels
kubectl get pods -l app=webapp
kubectl describe pod webapp-58958d9c46-zf5hs | grep -i image
```

### 61. Check the rollout history and make sure everything is ok after the update

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout

```
kubectl rollout history deployment webapp
```

### 62. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-undo-em-

```
kubectl rollout undo deployment webapp
kubectl describe pod webapp-5b94956f7-m8ql9 | grep -i image
```

### 63. Update the deployment with the image version 1.16.1 and verify the image and also check the rollout history
```
kubectl edit deployment webapp
kubectl get deployment webapp --show-labels
kubectl get pods -l app=webapp
kubectl describe pod webapp-64d7bdc68b-j6bhn | grep -i image
kubectl rollout history deployment webapp
```

### 64. Update the deployment to the Image 1.17.1 and verify everything is ok
```
kubectl rollout undo deployment webapp
kubectl rollout status deployment webapp
kubectl describe deploy webapp | grep -i image
```

### 65. Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment
```
kubectl edit deployment webapp
kubectl rollout status deployment webapp
kubectl get pod webapp-6d68d6984f-8jv8h
```

### 66. Undo the deployment with the previous version and verify everything is Ok
```
kubectl rollout undo deployment webapp
kubectl rollout history deployment webapp
kubectl rollout status deployment webapp
kubectl describe deployment webapp | grep -i image
```

### 66. Check the history of the specific revision of that deployment
```
kubectl rollout history deployment webbapp --revision=7
```

### 67. Pause the rollout of the deployment

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-undo-em-

```
kubectl rollout pause deployment webapp
```

### 68. Update the deployment with the image version latest and check the history and verify nothing is going on
```
kubectl edit deployment webapp
kubectl rollout history deployment webapp
```

### 69. Resume the rollout of the deployment
```
kubectl rollout resume deployment webapp
kubectl rollout history deployment webapp
kubectl describe deployment webapp | grep -i image
```

### 70. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1

https://kubernetes.io/blog/2016/07/autoscaling-in-kubernetes/
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#:~:text=The%20Horizontal%20Pod%20Autoscaler%20automatically,other%20application%2Dprovided%20metrics).

HPA - Horizontal Pod Autoscale

```
kubectl autoscale deployment webbapp --min=10 --max=20 --cpu-percent=50
kubectl get hpa
kubectl get deployments --show-labels
kubectl get pods -l app=webbapp
kubectl describe deployment webapp
```

### 71. Clean the cluster by deleting deployment and hpa you just created
```
kubectl delete deployment webapp
kubectl get deployments
kubectl delete hpa webapp
kubectl get hpa
```

### 72. Create a Job with an image node which prints node version and also verifies there is a pod created for this job

https://kubernetes.io/docs/concepts/workloads/controllers/job/

```
kubectl create job nodeversion --image=node -- node -v
kubectl get job -w
kubectl get pod
```

### 73. Get the logs of the job just created
```
kubectl logs nodeversion-q79cn
```

### 74. Output the yaml file for the Job with the image busybox which echos “Hello I am from job”
```
kubectl create job hello-job --image=busybox -o yaml --dry-run=client > output.yaml -- /bin/sh -c "echo 'Hello I am from job'"
```

### 75. Run the job and verify the job and the associated pod is created and check the logs as well
```
kubectl apply -f .\output.yaml
kubectl get jobs --show-labels
kubectl describe job hello-job
kubectl get pods -l controller-uid=c05624fe-e53c-45e2-821a-2ca098a8f8a6,job-name=nodeversion
kubectl logs nodeversion-q79cn
```

### 76. Delete the job we just created
```
kubectl delete job hello-job
kubectl get jobs
```

### 77. Create the same job and make it run 10 times one after one
```
kubectl explain jobs.spec
kubectl create job run-job --image=busybox -o yaml --dry-run=client > output.yaml -- /bin/sh -c "echo 'This is just a test'"
```
Add `completions: 10` to the yaml file created.

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: run-job
spec:
  completions: 10
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo 'This is just a test'
        image: busybox
        name: run-job
        resources: {}
      restartPolicy: Never
status: {}
```
```
kubectl apply -f output.yaml
kubectl describe job run-job
```

### 78. Watch the job that runs 10 times one by one and verify 10 pods are created and delete those after it’s completed
```
kubectl get job -w
kubectl get pods
kubectl delete job run-job
kubectl get pods
```