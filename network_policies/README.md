# Network Policies

https://kubernetes.io/docs/concepts/services-networking/network-policies/
https://banzaicloud.com/blog/network-policy/

## 1. How many network policies do you see in the environment? We have deployed few web applications, services and network policies. Inspect the environment.

```
kubectl get networkpolicy --all-namespaces
kubectl get pods -l name=payroll
```

Last line is to check with pod is affected by the networkpolicy

**Note**: netpol is the short name for networkpolicy

## 2. What type of traffic is this Network Policy configured to handle?

```
kubectl describe networkpolicy payroll-policy
```

## 3. Create a network policy to allow traffic from the 'Internal' application only to the 'payroll-service' and 'db-service' You might want to enable ingress traffic to the pod to test your rules in the UI.

* **Policy Name**: internal-policy
* **Policy Types**: Egress
* **Egress Allow**: payroll
* **Payroll Port**: 8080
* **Egress Allow**: mysql
* **MYSQL Port**: 3306

First we need to collect the labels of the pods to be added in the next script: `kubectl get pods --show-labels`

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
```

```
kubectl create -f output.yaml
kubectl describe netpol internal-policy
```