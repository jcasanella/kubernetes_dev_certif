# ConfigMaps

https://kubernetes.io/es/docs/concepts/configuration/configmap/

## 1. Get number of pods

`kubectl get pods`

## 2. What is the environment variable name set on the container in the pod?

`kubectl describe pod webapp-color`

and this is the content of the pod:

```
Name:         webapp-colorNamespace:    default
Priority:     0
Node:         node01/172.17.0.93
Start Time:   Wed, 26 Aug 2020 07:13:11 +0000
Labels:       name=webapp-color
Annotations:  <none>
Status:       Running
IP:           10.244.1.3
IPs:
  IP:  10.244.1.3
Containers:
  webapp-color:
    Container ID:   docker://8b2bdc111b8250909948223a1e292e123faff82b94a9ba9b0d99d4ddadca0e63
    Image:          kodekloud/webapp-color
    Image ID:       docker-pullable://kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 26 Aug 2020 07:13:28 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8vz94 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-8vz94:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8vz94
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  25s   default-scheduler  Successfully assigned default/webapp-color to node01
  Normal  Pulling    22s   kubelet, node01    Pulling image "kodekloud/webapp-color"
  Normal  Pulled     10s   kubelet, node01    Successfully pulled image "kodekloud/webapp-color"
  Normal  Created    9s    kubelet, node01    Created container webapp-color
  Normal  Started    8s    kubelet, node01    Started container webapp-color
```

## 3. Update the environment variable in the POD to display a green background. Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod

* Pod Name: web-app-color
* Label Name: webapp-color
* Env: APP_COLOR=green

`kubectl get pod -o=yaml webapp-color > pod.yaml`

and the content of the definition fie is:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-08-26T07:13:11Z"
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color
    name: webapp-color
```

Delete the pod

`kubectl delete pod webapp-color`

Edit the yaml file created and change the color to green

`kubectl apply -f output.yaml`

## 4. How many ConfigMaps exist in the environment

`kubectl get cm`

## 5. Identify the database host from the config map 'db-config'

`kubectl describe cm  db-config`

and the output is:

```
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_PORT:
----
3306
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01
Events:  <none>
```


## 6. Create a new ConfigMap for the webapp-color POD. Use the spec given below:

* ConfigName name: webapp-config-map
* Data: APP_COLOR=darkblue

We can start out from the definition of the existing ConfigMap

`kubectl get cm -o=yaml db-config > cm.yaml`

```
apiVersion: v1
data:
  DB_HOST: SQL01.example.com
  DB_NAME: SQL01
  DB_PORT: "3306"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-08-26T07:21:25Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:DB_HOST: {}
        f:DB_NAME: {}
        f:DB_PORT: {}
    manager: python-requests
    operation: Update
    time: "2020-08-26T07:21:25Z"
  name: db-config
  namespace: default
  resourceVersion: "2612"
  selfLink: /api/v1/namespaces/default/configmaps/db-config
  uid: 2e131e08-0d90-4d48-a95e-b7e472768b60
```

and we create the yaml like this one:

```
apiVersion: v1
data:
  APP_COLOR: darkblue
kind: ConfigMap
metadata:
  name: webapp-config-map
```

```
kubectl apply -f cm.yaml
kubectl get cm
```

other option is `kubectl create cm webapp-color --from-literal=APP_COLOR=darkblue`

## 7. Update the environment variable on the PD use the newly created ConfigMap. Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the pod

* Pod Name: webapp-color
* EnvFrom: webapp-config-map

`kubectl delete pod wbapp-color`

To get some help:

`kubectl explain pods --recursive | grep envFrom -A3`

Yaml:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
  resourceVersion: "1266"
  selfLink: /api/v1/namespaces/default/pods/webapp-color
  uid: cbf5e58e-5877-4683-a428-1debf160e98a
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
          configMapKeyRef:
              name: webapp-config-map
              key: APP_COLOR
    image: kodekloud/webapp-color
    name: webapp-color
    ```