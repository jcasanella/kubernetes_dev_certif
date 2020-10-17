# Jobs

https://kubernetes.io/docs/concepts/workloads/controllers/job/

## 1. Create a job with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

```
kubectl apply -f output.yaml
kubectl get jobs
```

Also can be created with the command:

```
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

## 2. Wait till it's done, get the output

```
kubectl get jobs -w
kubectl get pods
kubectl logs pi-hk4sx
kubectl delete job pi
```

## 3. Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

```
kubectl create job busybox  --image=busybox -- bin/sh -c 'echo hello; sleep 30; echo world'
kubectl get jobs
kubectl get job busybox --show-labels 
kubectl get jobs -l job-name=busybox
```

## 4. Follow the logs for the pod (you'll wait for 30 seconds)

```
kubectl get pods
kubectl logs busybox-kdm8t
```

## 5. See the status of the job, describe it and see the logs

```
kubectl get jobs -l job-name=busybox
kubectl describe job busybox
kubectl logs job/busybox
```

## 6. Delete the job

```
kubectl delete job busybox
kubectl get jobs -l job-name=busybox
```

## 7. Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

```
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
```

vi job.yaml add  `activeDeadlineSeconds: 30`

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busybox
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: Never
status: {}
```

```
kubectl apply -f job.yaml
```

## 8. Create the same job, make it run 5 times, one after the other. Verify its status and delete it

**Note**: Delete job from prev run: `kubectl delete job busybox`

```
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
```

vi job.yaml add `job.spec.completions=5`

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busybox
spec:
  completions: 5
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: Never
status: {}
```

```
kubectl get job busybox -w
kubectl delete job busybox
 ```

## 9. Create the same job, but make it run 5 parallel times

```
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
```

vi job.yaml add `job.spec.parallelism=5`

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busybox
spec:
  parallelism: 5
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: Never
status: {}
```

```
kubectl get job busybox -w
kubectl delete job busybox
```
