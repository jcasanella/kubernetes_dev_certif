# Deployments

* How to check if something exists: `kubectl get all`
* Create a deployment: `kubectl create -f deployment_file.yaml`
* Check status deployment: `kubectl rollout status deployment/my-app` - rollout is the process to create a deployment in the back end.
* Show revisions per deployment: `kubectl rollout history deployment/my-app`
* Delete a deployment: `kubectl delete deployment deployment/my-app`
* Create a deployment with track of the changes: `kubectl create -f deployment_file.yaml --record`
* How to update the version of a container in a deployment. We can do:

1. Change the version inside of the deployment file for the specific container
2. Run the command: `kubectl set image deployment/my-app nginx-container=nginx:1.12-perl`

* Rollback a change: `kubectl rollout undo deployment/my-app`