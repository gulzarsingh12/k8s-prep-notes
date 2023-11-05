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


# StatefulSet
Sometimes we need to create the deployment in such a way then we want to remember pod names in order. For example in database HA setup. Here we know that for example
for mysql. We can read from any replication node but can write to only one(master). This means that we need to remember which node is master. Pods dont have dns names and there ip addresses are dyanmic. Hence what could be the solution?
StatefulSets are used to create pods in ordered fashion. It will create pods in order and one by one. mysql-0,mysql-1.... and will scale up and down in same manner too. You can also set pod creation to parallel if you want. `spec.podManagementPolicy` can be updated to `Parallel` from defaul `OrderedReady`.

## Volumes
In stateful sets, volumes are not managed as deployment. it is done differently.
There is no separate PVC or PV creation. but PVC configuration is moved under `spec.volumeClaimTemplates` section. So pvc, pv creation is automatic for each pod. You can use the storage class to handle the volume creation and create volume for pods as each pod will have seprate pvc,pv and/or shared/dedicated volume deoending on storage class.

## Headless Service
A service is created with `clusterIP: None`. This is called headless service. Purpose of this service is to create dns A records for pods. As we know that pods needs to have dns names here instead of ip based FQDN names. 
Rememeber to set the `serviceName` property in StatefulSet's pod template. This is required to generate the unique dns names for each pod.
Below is th format of dns names of pod
<pod-name>.<svc-name>.<namespace>.svc.cluster.local
For example if serive name is `mysql-h` and pod name is `mysql` in default namespace then dns name of first pod will be `mysql-0.mysql-h.default.svc.cluster.local`. Still pod can be accessed via `mysql-h.default.svc.cluster.local` service dns name but chances depend on load balancer. 
For example in case of mysql dbs here, for any read operation `mysql-h.default.svc.cluster.local` can be used but for write `mysql-0.mysql-h.default.svc.cluster.local` should be used only.

# Admission Controller
Sometime we need to add additional control or security messures before creating pods or any resources. This can be done after authentication and authorization. 

client [kubectl] -> kube-apiserver [authentication -> authorization -> admission controller -> Create Pod]

So for example, 
- we want to ensure a user is always added to the pod created so that we know who created this pod.
- we want to add some label to every pod creation
- suppose we want to disable root for docker container at runtime.

This can be acheieved with admission controllers.

To check the enabled/diabled admission plugins
````
k exec -n kube-system -it kube-apiserver-controlplane -- kube-apiserver -h | grep enable-admission-plugins
````

To enable an admission controller set `--enable-admission-plugins=<admissions-ontroller>` in **kube-apiserver** manifest args
To disable an admission controller set `--disable-admission-plugins=<admissions-ontroller>`

## ValidatiingAdmissionController
This is the admission controller which validates the coming admissionreview request with allowed=true/false. if allowed it will go to next admission controller.

## MutatingAdmissionController
Sometimes we want to update the coming pod request with something then mutating will add/change the pod configurartion. for example if we want to add the non root user to container pod request if nothing specified. 

## Validate and Mutate
An admission controller can do both validate and mutate. for example, if requeet is coming without user,  mutating controller will add it as non root user, then validating controller will validate this request for the user field if specified as valid value.

Another example of validating is `NamespaceExists` and mutating is `NamespaceAutoprovision`. Please note that order of executiuon is always mutating and validating to ensure that request is updated already before validation otherwsie it will fail.

`NamespaceAutoprovision` and `NamespaceExists` is deprecated and replaced with `NamespaceLifecycle`. it will also ensure that user can't recreate the `default, kube-system and kube-public` namespaces.

## Configure admission webhook
Sometime we need to hook our own code to validate or mutate. This can be done via 2 webhooks.i.e MutatingAdmissinWebhook, ValidatingAdmissinWebhook. We need to configure to add our webhook server. Our webhook server will need to implement **/mutate** and **/validate** api post methods.


- To configure it,  we need to run our webhook server as systemd service or any other way externally or internally or as pod or deployment.
- Then we need to apply below configuration to enable the webhook.
````
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       "Namespaced"
  clientConfig:
    service:
      namespace: "example-namespace"
      name: "example-service"
    caBundle: <CA_BUNDLE>
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 5
````

Remember to set the caBundle. it is base64 encode ca.crt given by webhook server. You can openssl to generate certs for webshook server. which will have server.key,server.crt,ca.crt. Set the ca.crt as explained in above.

If running it as deployment/pod in same cluster, then set the webhook-service to reach the pods. As in above example, name should be like example-service.example-namespace so it can call it using that or FQDN.

# Api Versions
`k explain deployment` command to check version and other details of deployment resource.

there are 2 versions
- `preferred` version is recommended version which k8s want you to use. this contain latest features to use etc.
- `storage` version is the version in which data in stored in etcd. It can be same as preferred or different too. this is internal version in which k8s stores data in etcd.

To test a particular feature/version add `--runtime-config=batch/v2alpha1` to kube-apiserver args

## Deprecation
A version deprecated in a release must be supported for
- **GA**   : 12 months or 3 releases (whichever is longer)
- **beta** : 9 months or 3 releases (whichever is longer)
- **alpha**: 0 releases

use `kubectl-convert -f old.yaml --output-version v1` to migrate exiting version to new version.


# Custom Resources
Sometime we need to create to extend k8s and add our own resource apart from given k8s resources.i.e pods,deployments,services,ReplicaSets etc.

## CRD (CustomResourceDefinition)
- Create a CRD yaml file to create crd configuration
  ````
  apiVersion: apiextensions.k8s.io/v1
  kind: CustomResourceDefinition
  metadata:
    # name must match the spec fields below, and be in the form: <plural>.<group>
    name: crontabs.stable.example.com
  spec:
    # group name to use for REST API: /apis/<group>/<version>
    group: stable.example.com
    # list of versions supported by this CustomResourceDefinition
    versions:
      - name: v1
        # Each version can be enabled/disabled by Served flag.
        served: true
        # One and only one version must be marked as the storage version.
        storage: true
        schema:
          openAPIV3Schema:
            type: object
            properties:
              spec:
                type: object
                properties:
                  cronSpec:
                    type: string
                  image:
                    type: string
                  replicas:
                    type: integer
    # either Namespaced or Cluster
    scope: Namespaced
    names:
      # plural name to be used in the URL: /apis/<group>/<version>/<plural>
      plural: crontabs
      # singular name to be used as an alias on the CLI and for display
      singular: crontab
      # kind is normally the CamelCased singular type. Your resource manifests use this.
      kind: CronTab
      # shortNames allow shorter string to match your resource on the CLI
      shortNames:
      - ct
  ````
- `k apply -f <crdfile>`

## CustomResource
- create resource like below
  ````
  apiVersion: "stable.example.com/v1"
  kind: CronTab
  metadata:
    name: my-new-cron-object
  spec:
    cronSpec: "* * * * */5"
    image: my-awesome-cron-image
  ````
- Access this resource `k get ct` or `k get crontabs`


