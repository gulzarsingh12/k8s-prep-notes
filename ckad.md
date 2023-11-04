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

The back-off limit is set by default to 6. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s ...) capped at six minutes.

## Cron Jobs
Jobs which are scheduled to run at scheduled time. it will have `startDeadlineSeconds`, which means if the job is not started within this time after missing the schedule then it will skipped.

`schedule` is defined as below
- `* * * * *`  starts every min
- `0 * * * *`  starts every hour
- `0 0 * * *`  start midnight every day
-  `0 0 1 * *` start every month 1st day at midnight
-  `0 0 1 1 *` start every year 1st day of 1st month at midnight
-  `0 0  * * 0` start every week on first day of week at midnight

*/2 for mins means every other min, */5 for mins means every 5th mins.

`successfulJobsHistoryLimit` keeps history of successful jobs : 3 default
`failedJobsHistoryLimit` keeps history of failed jobs: 1 default

           
