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

# Labels & Selectors
## Labels
- To get the pods using label selector
  ````
  k get po -l env=dev
  ````
- `-l or --selector` can be used to select the labels.
- To apply filter you can use `in for multiple values` like
  ````
  k get po -l 'env in (dev,prod)'
  ````
- Multiple labels
  ````
  k get po -l env=prod,bu=finance,tier=frontend
  ````
- To create a pod with label
  ````
  k run nginx --image=nginx -l=app=frontend
  ````
- To show labels on pods or nodes etc
  ````
  k get po --show-labels
  ````
  
# Taints & tolerations
Taint is defined on noded and tolerations on pod to allow nodes the pods with tolerations.
- `k taint node node01 spray=mortein:NoSchedule` to create taint on node01
- ````
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
  ````
  to creat toleration on pod.
- To remove the taint on node `k taint node node01 spray=mortein:NoSchedule-` simply add - (dash)

# NodeSelector
- add below to the pod
  ````
  nodeSelector:
    size: large
  ````
  label `size=large` should exist on node to schedule the above pod on this node.
- However it can't handle complex usecase
  - like size large or medium
  - not small
  For this, use nodeAffinitiy
- `NoSchedule` means not to schedule any pod after the taint is applied but existing pod will be running
- `NoExecute` means no pods allowed without tolerations. if a pod is running without toleration then it will be evicted.
- `PreferNoSchedule` is a "preference" or "soft" version of NoSchedule. The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, but it is not guaranteed.

# Affinity
## NodeAffinity
similar to nodeselector but this is more pwerful because it provides filter 
````
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd  
````
so it wil place the pod on node with disktype in ssd.

## PodAffinity
It is interpod affinity and tell to schedule pods on same node which has existing pods running. It means for below to run pod on the node having `topologyKey` specified label with existing pod having label `app=store`.
````
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
             - store
        topologyKey: "kubernetes.io/hostname"
````
## PodAntiAffinity
It is the opposite of the pod affinity. It will schedule the pod on the node which doesn't have the pod already with matching label already. Again it is also applicable to node with label as toplogyKey.
This is useful when we need to colocate the pods on the nodes. This is good example for apps using caching to avoid the hit/latency going to different pod..i.e. redis cache.. e.g. zookeeper link in k8s documentation.
````
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - store
      topologyKey: "kubernetes.io/hostname"
````

### pod affinity and anti affinity together
It will be useful to create a set with only 1 webserver (anti) located with redis cache (at least 1 to colocate)
````
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - web-store
      topologyKey: "kubernetes.io/hostname"
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - store
      topologyKey: "kubernetes.io/hostname"
````

# ToplogySpreadConstraint
This is to schedule pods with equal replicas on each node. for example if there are 20 replicas and 4 nodes. it will ensure 5 pods on each node. Based on the label it will count the no of pods to check the imbalance (maxSkew).
````
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        foo: bar
````

# Resource Requirements
- there are 2 type of resource request limits
- 1. requests. 2. limits
- request is what you are requesting and limit is what the max limit

As cluster is shared between different teams because nodes are deployed with the cluster. To constraint the resource usage for individual teams, `ResourceQuota` is defined at namespace level. So resource can be maxed out till the namespace resource quota limit but it cant take resource for other namespaces. 

However, there is still a problem exists in the namespace level. What if some container/pods take all the namespace level resources? This can be solved by setting the limits at container level. `LimitRange` is used to set the default, defaultRequest, min, max etc at container level.

## Pod termination
When a node is low on resources and need to terminate a resource then it will work as below:

It will check the pods using more cpu and memory then the requested (`requests:`) resources. it will terminate those pods. For example there are 3 pods with cpu requests 2m, 1m and not defined (this is like 0m). K8s will terminate the pod with cpu 0m as this is resource using more cpu the requested (0m, not defined). Hence its always good to define the requests for cpu and memory.

There are 2 ways to check the from kubectl
- Check the pods with least cpu or memory. `k get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].resources}{"\n"}{end}'`
- Check the QosClass (QualityOfService) . It can be like BestEffort, Burstable
  `k get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.qosClass}{"\n"}{end}'`
  Hence BestEffort will be terminated first 

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


leaderElection:
  leaderElect: false
````

To use the scheduler, set `schedulerName` in the pod yaml file. If scheduler is not configured correctly then, pod will remain in pending state. You can see scheduler name in events too.

to debug scheduler, view the logs of scheduler. if pod, then k logs <podname>

## PriorityClass
To set the prioroty of the pod, set property `priorityClassName` of the pod. To define the priority class, use below
````
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: Never   #Never,PreemptLowerPriority
````

**globalDefault** can be set to only one class. if set, all pods without priority class name will have this priority class name set to the pod.
preemptionPolicy set to Never means, it will wait for resources to be free for scheduling the pod but not preempt any existing pod. In case of PreemptLowerPriority, it will preempt the lower priority pods. This is the default existing behaviour too.

## How schedulking works
There are 4 phases which run in scheduling

### Scheduling process

**Scheduling Queue -> Filtering -> Scoring -> Binding**

#### Scheduling Queue
Pod will be put into a queue to schedule. **PrioritySort** plugin can be used for this.

#### Filtering
There can be multiple plugins to filter the nodes to remove unfit nodes (**NodeResourcesFit**).
Some other plugins are **NodeName**, **NodeUnschedulable**, **TaintsTolerations**, **NodeAffinity**, **NodePorts**
It will filter nodes based on tains, toleration, affinity, cpu. memory etc also.

#### Scoring
After filtering, nodes elgibile for scheduling will be scored to find the best node for scheduling.
some plugins are like **NodeResourcesFit**, **ImageLocality**
If pod needs 10 cpus, 1 node have 12 cpu and another have 16 cpu. Then node with 16 cpu will have higher score/
if some node have docker image already pulled for the pod container to run, it can be scored higher.

#### Binding
Once best node is identified, it will be bind to the pod.
plugin can be **DefaultBinder**

### Extension points
Each phase have extenions points, these extension points can be used to extend the scheduling.
This will help use the same scheduler with multiple profiles. since v1.18, we can use these extensions to use specific plugins at each extension point. we can enable/disable and attached custom plugins too. This can be set under profiles in scheduler configuration.
Below are extension points

Scheduing Queue :  **queueSort**
Filtering: **preFilter**, **filter**, **postFilter**
Scoring: **preScore**, **score**, **reserve**, **permit**
Binding: **preBind**, **bind**, **postBind**

reserve is done before binding to avoid race conditions to reserve the resources for pod on a node.
permit - once plugins in permit approve the pod, it is sent to binding.

### Scheduling profiles
As mentioned above, with extension points, its possible to customnize the beahaviour of scheduling as per requirement instead of creating multiple schedulers. See below, how to use multiple schedulers using profiles
````
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler1
    plugins:
      score:
       disabled:
        - name: TaintTolertion
       enabled:
        - name: MyCustomPluginA
        - name: MycustomerPluginB
  - schedulerName: my-scheduler2
    plugins:
      preScore:
       disabled:
        - name: '*'
      score:
       enabled:
        - name: '*'
````
