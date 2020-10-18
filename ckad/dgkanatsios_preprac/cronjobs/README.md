# Cronjobs

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

## 1. Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

other option 

```
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

## 2. See its logs and delete it

```
kubectl get pods
kubectl logs busybox-1603039080-hjd94
kubectl delete cronjob busybox
```

## 3. Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its schedule.

```
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' --dry-run=client -o yaml > output.yaml
```

Edit file and add the following entry `startingDeadlineSeconds: 17 `

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busybox
spec:
  startingDeadlineSeconds: 17
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busybox
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: busybox
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
```

```
kubectl apply -f output.yaml
```