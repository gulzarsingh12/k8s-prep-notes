# Deployment

## Deployment strategy

### Recreate
Kubernetes strategy to delete all pods and recreate all the pods. In this strategy, all pods will be terminated, hence application wont be available 
during deployment.

### Rolling Update
This is the default kubernetes strategy where pods are terminated and recreated at the besed on the max unavailable and max surge. For example if 
deployment has set 4 replicas and max unavail and surge is 25% then updated are rolled by terminating 1 pod first and then bring up 1 pod. Once 1 
pod is up then 2 is termibated and recreated.
This will ensure no application downtime hence default deployment strategy.

### Blue Green
In this strategy, new version version is deployed and tested. if working, then old deployment is terminated. So in this case, if 4 pods are deployed 
then 4 new pods are created totalling 8 pods. 

### Canary
In this deployment, new changes are rolled with old one and tested to ensure it is working if working fine then new changes are deployed rolled out slowely to all pods.


# Jobs 
job will run to completin. if job is failed, it will restart to retry and rerun. it will retry till `backoffLimit` which is default to `6`. Sometimes it is possible to set `activeDeadlineSeconds` which will kill the job even if `backoffLimit` is avilable to retry few more time. Hence activeDeadline has precendence over backoffLimit.

By default `completions` is set to 1 but if set to >1 then it will ensure the job is completed that many times. it will try to do completions sequentially one by one. 
To try these completions parallely, `parallelism` can be set to try more than 1 completions. it is also set to 1 by default.

