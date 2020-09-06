# Multicontainers

https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

## Some theory

https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html

### Sidecar pattern

The sidecar pattern consists of a main application—i.e. your web application—plus a helper container with a responsibility that is essential to your application, but is not necessarily part of the application itself.

The most common sidecar containers are logging utilities, sync services, watchers, and monitoring agents. It wouldn't make sense for a logging container to run while the application itself isn't running, so we create a pod that has the main application and the sidecar container. Another benefit of moving the logging work is that if the logging code is faulty, the fault will be isolated to that container—an exception thrown in the nonessential logging code won't bring down the main application.

```
# Example YAML configuration for the sidecar pattern.

# It defines a main application container which writes
# the current date to a log file every five seconds.

# Once the pod is running:
#   
#   (Connect to the sidecar pod)
#   kubectl exec pod-with-sidecar -c sidecar-container -it bash
#   
#   (Install curl on the sidecar)
#   apt-get update && apt-get install curl
#   
#   (Access the log file via the sidecar)
#   curl 'http://localhost:80/app.txt'

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sidecar
spec:
  # Create a volume called 'shared-logs' that the
  # app and sidecar share.
  volumes:
  - name: shared-logs 
    emptyDir: {}

  # In the sidecar pattern, there is a main application
  # container and a sidecar container.
  containers:

  # Main application container
  - name: app-container
    # Simple application: write the current date
    # to the log file every five seconds
    image: alpine 
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/app.txt; sleep 5;done"]

    # Mount the pod's shared log file into the app 
    # container. The app writes logs here.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # Sidecar container
  - name: sidecar-container
    # Simple sidecar: display log files using nginx.
    image: nginx:1.7.9
    ports:
      - containerPort: 80

    # Mount the pod's shared log file into the sidecar
    # container. In this case, nginx will serve the files
    # in this directory.
    volumeMounts:
    - name: shared-logs
      mountPath: /usr/share/nginx/html # nginx-specific mount path
```

### Adapter pattern

The adapter pattern is used to standardize and normalize application output or monitoring data for aggregation.

As a simple example, we have a cluster-level monitoring agent that tracks response times. Say we have a Ruby application in our cluster that writes request timing in the format [DATE] - [HOST] - [DURATION], while another Node.js application writes the same information in [HOST] - [START_DATE] - [END_DATE].

The monitoring agent can only accept output in the format [RUBY|NODE] - [HOST] - [DATE] - [DURATION]. We could force the applications to write output in the format we need, but that burdens the application developer, and there might be other things depending on this format. The better alternative is to provide adapter containers that adjust the output into the desired format. Then the application developer can simply update the pod definition to add the adapter container and they get this monitoring for free.

```
# Example YAML configuration for the adapter pattern.

# It defines a main application container which writes
# the current date and system usage information to a log file 
# every five seconds.

# The adapter container reads what the application has written and
# reformats it into a structure that a hypothetical monitoring 
# service requires. 

# Once the pod is running:
#   
#   (Connect to the application pod)
#   kubectl exec pod-with-adapter -c app-container -it sh
# 
#   (Take a look at what the application is writing.)
#   cat /var/log/top.txt
#   
#   (Take a look at what the adapter has reformatted it to.)
#   cat /var/log/status.txt


apiVersion: v1
kind: Pod
metadata:
  name: pod-with-adapter
spec:
  # Create a volume called 'shared-logs' that the
  # app and adapter share.
  volumes:
  - name: shared-logs 
    emptyDir: {}

  containers:
  # Main application container
  - name: app-container
    # This application writes system usage information (`top`) to a status 
    # file every five seconds.
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do date > /var/log/top.txt && top -n 1 -b >> /var/log/top.txt; sleep 5;done"]

    # Mount the pod's shared log file into the app 
    # container. The app writes logs here.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # Adapter container
  - name: adapter-container
    # This sidecar container takes the output format of the application
    # In this example, our monitoring service requires status files
    # to have the date, then memory usage, then CPU percentage each 
    # on a new line.
    
    # Our adapter container will inspect the contents of the app's top file,
    # reformat it, and write the correctly formatted output to the status file.
    image: alpine
    command: ["/bin/sh"]

    # A long command doing a simple thing: read the `top.txt` file that the
    # application wrote to and adapt it to fit the status file format.
    # Get the date from the first line, write to `status.txt` output file.
    # Get the first memory usage number, write to `status.txt`.
    # Get the first CPU usage percentage, write to `status.txt`.

    args: ["-c", "while true; do (cat /var/log/top.txt | head -1 > /var/log/status.txt) && (cat /var/log/top.txt | head -2 | tail -1 | grep
 -o -E '\\d+\\w' | head -1 >> /var/log/status.txt) && (cat /var/log/top.txt | head -3 | tail -1 | grep
-o -E '\\d+%' | head -1 >> /var/log/status.txt); sleep 5; done"]


    # Mount the pod's shared log file into the adapter
    # container.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

### Ambassadpr pattern

The ambassador pattern is a useful way to connect containers with the outside world. An ambassador container is essentially a proxy that allows other containers to connect to a port on localhost while the ambassador container can proxy these connections to different environments depending on the cluster's needs.

One of the best use-cases for the ambassador pattern is for providing access to a database. When developing locally, you probably want to use your local database, while your test and production deployments want different databases again.

Managing which database you connect to could be done through environment variables, but will mean your application changes connection URLs depending on the environment. A better solution is for the application to always connect to localhost, and let the responsibility of mapping this connecting to the right database fall to an ambassador container. Alternatively, the ambassador could be sending requests to different shards of the database—the application itself doesn't need to worry.

## 1. Identify the number of containers running in the 'red' pod.

```
kubectl get pods
kubectl describe pod red | grep -i 'containers' -A 50
```

## 2. Identify the name of the containers running in the 'blue' pod.

```
kubectl get pods
kubectl describe pod blue | grep -i 'containers' -A 50
```

## 3. Create a multi-container pod with 2 containers.

* Name: yellow
* Container 1 Name: lemon
* Container 1 Image: busybox
* Container 2 Name: gold
* Container 2 Image: redis

`kubectl run yellow --image busybox --dry-run=server -o yaml > output.yaml`

edit output.yaml to add the second container and change the name of the first one.

```
  containers:
  - image: busybox
    imagePullPolicy: Always
    name: lemon
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-gbvgh
      readOnly: true
  - image: redis
    name: gold

kubectl apply -f output.yaml
```
## 4. We have deployed an application logging stack in the elastic-stack namespace. Inspect it.

```
kubectl get ns elastic-stack
kubectl describe ns elastic-stack
```

## 5. Inspect the 'app' pod and identify the number of containers in it. It is deployed in the elastic-stack namespace

`kubectl --namespace=elastic-stack describe pod app`

## 6. The 'app'lication outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login. Inspect the log file inside the pod

`kubectl --namespace=elastic-stack exec -it app -- cat /log/app.log | more`

## 7. Edit the pod to add a sidecar container to send logs to ElasticSearch. Mount the log volume to the sidecar container. Only add a new container. Do not modify anything else. Use the spec on the right.

* Name: app
* Container Name: sidecar
* Container Image: kodekloud/filebeat-configured
* Volume Mount: log-volume
* Mount Path: /var/log/event-simulator/
* Existing Container Name: app
* Existing Container Image: kodekloud/event-simulator

```
kubectl --namespace=elastic-stack get pod app -o yaml > output.yaml
kubectl --namespace=elastic-stack delete pod app 

spec:
  containers:
  - image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: app
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /log
      name: log-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-dcbhh
      readOnly: true
  - image: kodekloud/filebeat-configured
    name: sidecar
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

kubectl apply -f output.yaml
```