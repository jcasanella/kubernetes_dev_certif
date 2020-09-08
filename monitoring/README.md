# Monitoring Containers

https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

## 1. We have deployed a few PODs running workloads. Inspect it.

```
kubectl get pods
kubectl get pod elephant
kubectl get pod elephant -o wide
kubectl get pod lion -o wide
kubectl get pod rabbit -o wide
```

## 2. Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.

https://github.com/kodekloudhub/kubernetes-metrics-server.git

```
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
```

## 3. Deploy the metrics-server by creating all the components downloaded. Run the 'kubectl create -f .' command from within the downloaded repository.

```
cd kubernetes-metric-server
kubectl create -f .
```

## 4. It takes a few minutes for the metrics server to start gathering data. Run the 'kubectl top node' command and wait for a valid output.

## 5. Identify the POD that consumes the most Memory.

`kubectl top pod`

