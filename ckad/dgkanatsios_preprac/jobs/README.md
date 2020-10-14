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