# Labels and Selectors

https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/

## 1. We have deployed a number of PODs. They are labelled with 'tier', 'env' and 'bu'. How many PODs exist in the 'dev' environment? Use selectors to filter the output

```
kubectl get pods --selector env=dev | tail -n +2 | wc -l
```

## 2. How many PODs are in the 'finance' business unit ('bu')?

```
kubectl get pods --selector bu=finance | tail -n +2 | wc -l
```

## 3. How many objects are in the 'prod' environment including PODs, ReplicaSets and any other objects?

```
kubectl get all --selector env=prod
```

## 4. Identify the POD which is 'prod', part of 'finance' BU and is a 'frontend' tier?

```
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

## 5. A ReplicaSet definition file is given 'replicaset-definition-1.yaml'. Try to create the replicaset. There is an issue with the file. Try to fix it.

* ReplicaSet: replicaset-1
* Replicas: 2

`kubectl apply -f replicaset-definition-1.yaml`

File:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
      - name: nginx
        image: nginxmaster
```

Change label: `tier: nginx` to `tier: frontend`