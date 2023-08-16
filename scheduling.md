# Manual Scheduling
Suppose there is no scheduler set up. Then pods will be stuck in pending state.To schedule the pod manually when no
scheduler exists, we can set the **nodeName** property of pod definition file. 

But this is only possible for new pods creation. What if a pod is already created and we want to schedule it. This
can be done with Binding. You can apply below yaml from kubectl or submit rest api call


## binding.yaml
````
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node01
````

## cli
````
k run nginx --image=nginx
k apply -f binding.yaml
````

## rest
Only binding part as below. created pod from cli. Also get the certs from `k config view`
````
curl -k --cert /root/cert.pem \
--cacert /root/cacert.pem \
--key /root/key.pem \
--header "Content-Type:application/json" \
--request POST \
--data '{ "apiVersion": "v1", "kind": "Binding", "metadata": { "name": "nginx" }, "target": { "apiVersion": "v1", "kind": "Node", "name": "node01" }}' \
https://controlplane:6443/api/v1/namespaces/default/pods/nginx/binding/
````

# DaemonSets
Daemonsets are like replicaSet but but instead of ensuring no of replicas, it will ensure 1 pod on each node. We can try to create rs using imperative command then replace kind with DeamonSet

There are many use cases of daemonsets, one of the usecase is kube-proxy which k8s does. another could networking. weavenent install an agent on each node. 
For user application, a monitoring agent or logviewer can be deployed on each node. this can ensure that agent is running if application pod is not and we want to taker some action or any other monitoring depending on use case.

## How it works
Till v1.12, it used to set `nodeName` on each pod. since 1.12 it is setting `nodeAffinity` on each **pod**

# Static Pods
What happen when worker node is on its own? There is no k8s cluster. Kubelet is on its own to manage pods.

Kubelet will acually monitor a filesystem directory  periodically to bring up/down pods and if pod yaml is added into this directory, it will create a pod. if file is removed from the directory, pod is removed. 

When a pod is created, a suffix with node name is added to the pod name. For example, when kube-apiserver pod is created. It will create it with name `kube-apiserver-controlplane` on current node with name `controlplane`

These are readOnly pods, can't be deleted from kubectl. can only be deleted by deleting the yaml from configured directory.

Note that you can only create pods not deployment or replicaset.

## pod yaml directory

There are 2 ways to specify the directory for pod yaml files

### directly to the kubelet service
Pass the option pod-manifest-path in kubelet.service for kubelet
`--pod-manifest-path=/etc/kubernetes/manifests`

### kubelet config
Set the option staticPodPath in config of kubelet.service.
`--config=kubeconfig.yaml` in  kubelet.service
then in kubeconfig.yaml define `--staticPodPath=/etc/kubernenets/manifests`

Keep this in mind when debugging to check if podManifestPath and then config if not podMainfestPath found.

There is no kubectl so docker ps can be used to view containers.

# Multiple Scheduler
We can configure custom scheduler if default scheduler can't work for some usecase. we can configure multiple scheuler with unique name. Default (kube-scheduler) is named as `default-scheduler`. 

We can write our own scheduler and deploy it as service or pod/deployment. We need to set scheduler configuration and set it in options passed to scheduler.
https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

````
--config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
````

## my-scheduler-config.yaml
````
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: false
````

To use the scheduler, set `schedulerName` in the pod yaml file. If scheduler is not configured correctly then, pod will remain in pending state. You can see scheduler name in events too.

to debug scheduler, view the logs of scheduler. if pod, then k logs <podname>
