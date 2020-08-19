1. How many Namespaces exist on the system?

`kubectl get ns`

2. How many pods exist in the 'research' namespace?

`kubectl --namespace=research get pods`

3. Create a POD in the 'finance' namespace. Use the spec given on the right.

. name: redis
. image name: redis

`kubectl --namespace=finance run redis --image=redis`

4. Which namespace has the 'blue' pod in it?

`kubectl get pods --all-namespaces   | grep 'blue'`

5. What DNS name should the Blue application use to access the database 'db-service' in its own namespace - 'marketing'.

`<service-name>.<namespace-name>.svc.cluster.local`