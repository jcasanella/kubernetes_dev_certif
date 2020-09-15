# Jobs and Cronjobs

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
https://kubernetes.io/docs/concepts/workloads/controllers/job/

## 1. A pod definition file named throw-dice-pod.yaml is given. The image 'throw-dice' randomly returns a value between 1 and 6. 6 is considered 'success' and all others are 'failure'. Try deploying the POD and view the POD logs for the generated number.

* file located at /root/throw-dice-pod.yaml
* Pod Name: throw-dice-pod
* Image Name: kodekloud/throw-dice

```
kubectl apply -f throw-dice-pod.yaml
kubectl get pods
kubectl logs throw-dice-pod
```

## 2. Create a Job using this POD definition. Look at how many attempts does it take to get a '6'.

* Job Name: throw-dice-job
* Image Name: kodekloud/throw-dice

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: throw-dice-pod
        image: kodekloud/throw-dice
      restartPolicy: OnFailure
```

Other approach: `kubectl create job throw-dice-job --image kodekloud/throw-dice`

in our case run the following command to generate the yaml file and add 

```
kubectl create job throw-dice-job --image kodekloud/throw-dice --dry-run=client -o yaml > output.yaml
```
`backoffLimit: 4` at the same level as spec. 'kubectl create -f output.yaml'

Check if job finished:

`kubectl get jobs`


## 3. Monitor and wait for the job to succeed. Throughout this practice test remember to increase the 'BackOffLimit' to prevent the job from quitting before it succeeds. Check out the documentation page about the BackOffLimit property.

* Job Name: throw-dice-job
* Image Name: kodekloud/throw-dice
* Job Succeeded: True

There are situations where you want to fail a Job after some amount of retries due to a logical error in configuration etc. To do so, set `.spec.backoffLimit` to specify the number of retries before considering a Job as failed. The back-off limit is set by default to 6. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s ...) capped at six minutes. The back-off count is reset when a Job's Pod is deleted or successful without any other Pods for the Job failing around that time.

`kubectl describe jobs throw-dice-job`

## 4. Update the job definition to run as many times as required to get 3 successful 6's. Delete existing job and create a new one with the given spec. Monitor and wait for the job to succeed.

* Job Name: throw-dice-job
* Image Name: kodekloud/throw-dice
* Completions: 3
* Job Succeeded: True

`kubectl delete job throw-dice-job`

and edit to 

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
  backoffLimit: 25
  completions: 3
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

restart policy values:

* Always
* OnFailure
* Never

```
kubectl create -f output.yaml
watch "kubectl get jobs"
```

## 5. That took a while. Let us try to speed it up, by running upto 3 jobs in parallel. Update the job definition to run 3 jobs in parallel.

* Job Name: throw-dice-job
* Image Name: kodekloud/throw-dice
* Completions: 3
* Parallelism: 3

`kubectl delete job throw-dice-job` add parallelism to the spec level

```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
  parallelism: 3
  backoffLimit: 25
  completions: 3
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

`kubectl create -f output.yaml`

**Note**: Remember if you need to check the help you can run this command: `kubectl explain job --recursive`

## 6. Let us now schedule that job to run at 21:30 hours every day. Create a CronJob for this

* CronJob Name: throw-dice-cron-job
* Image Name: kodekloud/throw-dice
* Schedule: 30 21 * * *

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  schedule: "30 21 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: throw-dice-pod
            image: kodekloud/throw-dice
          restartPolicy: OnFailure
```

Other option `kubectl create cronjob throw-dice-cron-job --image kodekloud/throw-dice --schedule "30 21 * * *" --dry-run=client -o yaml > output2.yaml`

Create the job: `kubectl create -f output2.yaml` and check the cronjob `kubectl get cronjobs`

We can do the same using run command: (deprecated, better to use `kubectl create`)

```
kubectl run throw-dice-cron-job --schedule="30 21 * * *" --restart=OnFailure --image=kodekloud/throw-dice
```