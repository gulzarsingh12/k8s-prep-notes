# Introduction
Self notes to prepare for CKA exam

# Cluster Architecture
There are 2 types for nodes used in k8s cluster.
- master (for hosting <b>controlplane</b> components)
- worker (for hosting <b>user</b> applications)

## Master Node
### Etcd
This is key/value store type database to store the information about the k8s cluster. Every information about the cluster is stored in key value form in etcd. Only kube-apiserver will commonicate with etcd. Any other components.i.e. kube-schedular, controller-manager etc. will communicate through kube-apiserver for anything. There can be multiple instances of etcd for HA.

### Controller-mannager
This is called the brain of the k8s cluster. it will have controllers to perform the various operations in the cluster. i.e. Node Controller, Replication Controller etc. Node controller to add nodes, monitor nodes etc. Replication controller to ensure desired no of pods/containers are running.

### Kube-Scheduler
Scheduler will schedule the applications on the right node based on the parameter like taints, tolerartionbs, affinity etc on all nodes and like cpu, memory or requested capacity etc.

### Kube-Apiserver
This is the primary component to manage all the communication between controlplane components as well as with clients through rest api.

## Worker Node
### Kubelet
It is an agent running on each node. it will periodicaally fetch status from api server and run the containers based on the informartion. it will also give back the information on the status and health o the cluster.

### Kube-proxy
A pod can be scheuled on any worker node. So how to enable communication between containers/pods across the nodes? Kube proxy does this. it will define the network rules to ensure this.

## CRE(Container runtime Interface. e.g.Docker, ContainerD or rkt)
This is required to run the controlplane(components deployed as containers) and user workload on worker nodes.

# Container Runtime Concepts
## OCI (Open container Initiative)
This is a standard created by multiple leaders including docker to standardize how containers should build, run and distribute.
- Image Spec: Image spec consists of image configuration, image manifest and filesystem(layer) serialization. Any container runtime who confirms to OCI need to adhere to this.
- Runtime Spec: It will define how to run the image/fliesystem unpacked on the disk.
- Distribution Spec: to standardize the distribution of container images

## CRI (Container Runtime Interface)
Initially, K8s only supported the docker container runtime. But later other runtimes want to be run on K8s, so K8s introduced CRI to allow other CRE's to run containers on K8s as long as they follow OCI.

## CRE (Container Runtime Engine)
There are many runtime engines which are OCI based.e.g. Rkt, ContainerD etc.

### ContainerD
Initially K8s run on docker but after introducing the CRI, they created a workaround to support docker as docker wasn't conformant to CRI. K8s created dockershim for this.

Note that docker contains lot of tools including cli, build, volumes, runC etc. runC is the ontainer runtime which run containers and ContainerD is the daemon to run runC.

However others moved to CRI and Docker separated the runC + containerD too. Due to significant effort to support dockershim, K8s dropped it in v1.24. containerD was conformant to CRI so it could run containers on K8s.

containerD is a separte project and maintained by containerD/docker community.

## container CLI tools
### containerD
#### ctr
Ctr is the cli tool distributed with containerD. However, it has limited set of features. It can be used for debugging purposes. 

Some of the commands are:
````
ctr images pull docker.io/library/redis:alpine

ctr run docker.io/library/redis:alpine redis
````

#### nerdctr
A better alternative to ctr is nerdctr. This is general purpose tool and supports almost all commands as docker cli.
- supports docker compose
- supports newest containerD features like
  - encryptiing images
  - lazy pulling
  - P2P image distribution
  - image sigining and verifying
  - namespace in K8s
 
Same commands as docker just replace docker with nerdctr. for example
````
docker run --name redis -po 8282:8080 -e=Key=Value redis
nerdctr run --name redis -po 8282:8080 -e=Key=Value redis
````

### K8s
ctr and nerdctr can run docker containers but they are limited to containerD. what about CRE's running on K8s using CRI. K8s community worked on a tool called <b>crictl</b> which can work for any CRE which support CRI.

#### crictl
This is the tool maintained by K8s community to provide the cli for CRI based runtimes. It will work across runtimes.i.e. Rkt, containerD etc.
Again, this tool provides the limited set of features for debugging purposes. 

Note that if you run a container with crictl, then kubelet will kill it as it has no idea of anyone creating containers outside of its knowledge. Hence its recommended to use this tool for debug purposes only. 
It can be used to debug in case kubectl is not avalable to inspect logs, running containers and pods etc.

It is installed separately.

Some of the commands are:
````
crictl images
crictl pull busybox
crictl exec -i -t rwergfrwe32324
crictl logs rwergfrwe32324
crictl pods
````

Note: after v1.24, cri runtime endpoints were updated to remove dockershim.sock. dockershim.sock was replaced with cri-dockerd.sock. Now users are encouraged to manually set runtime and image endpoint.
````
crictl --runtime-endpoint
export CONTAINER_RUNTIME_ENDPOINT=unix:///host/run/containerd/containerd.sock
````

# ETCD
etcd is a key value store. it is like an object/document stored in json format. each object is stored as file hence change in one object doesn't affect another object.

RAFT protocol is used for ha/distributed cluster.RAFT uses RPC commands to do various operation like RequestVote, AppendEntry, Leader election etc. election timeout is randomly set to avoid split vote.

etcdctl is cli for interacting with etcd cluster. default api version is 2. Which uses set to set objects. while in v3 put is used to set objects. other notable difference is to get version. in v3, version is a command.

etcd can be downloaded and deployed standalone from github. 
if deployed via kubeadm, it is deployed as pod in kube-system namespace.

every information about cluster is stored in etcd.i.e. pods,nodes,config,secrets, accounts, roles, bindings etc.
Any change is marked completed after it is persisted to etcd.

````
ETCDCTL_API=3 ./etcdctl version
````

ETCD is run on 2379 port. below can be set as ip address for kube apiserver to reach etcd.
````
--advertise-client-urls https://${INTERNAL_IP}:2379
````

To connect to etcd cli, one way is to exec into pod and run etcdctl
````
k exec etc-master -n kube-system etcdctl get / --prefix --keys-only
````

Set initial-cluster command line option to set etcd service for ha.
````
--initial-cluster controller-0=https://${CONTROLLER0_IP}:2379,controller-1=https://${CONTROLLER1_IP}:2379
````

## commands
To work with etcdctl
````
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
````
Note that path of certs is /etc/kubernetes/pki/etcd . to specify the certs use option --cert, --key, --cacert

api v3
````
etcdctl snasphsot save  (to take backup)
etcdctl endpoint health
etcdctl get
etcdctl put
````

# Kube Apiserver
When a command is executed on kubectl, it is recieved by apiserver and following steps are performed. for example a request to create a pod.
1. it will authenticate the user
2. it will validate the request.
3. it will update the etcd cluster with pod creation request
4. It will return to pod creation request.
5. kube-schedular will listen to apiserver and see the request for pod creation. It will assess the various node parameters to schedule the pod. it will return back with pod scheduling information to api server.
6. Api server will update etcd cluster and inform the kubelet to run the containers for pod
7. kubeket will then pass the instrcution to CRE to create the containers. Once created, it will update back to api server
8. api server will update the etcd.

The above same pattern is followed for almost every request.

apiserver will set the tls for etcd and kubelet ad below
````
--etcd-servers=https://127.0.0.1:2379
--etcd-cafile=/var/lib/kubernetes/ca.pem
--etcd-certfile=/var/lib/kubernetes/kubernetescert.pem
--etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem
--kubelet-certificate-authority=/var/lib/kubernetes/ca.pem
--kubelet-client-certificate=/var/lib/kubernetes/kubernetescert.pem
--kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem
--kubelet-https=true
````

to view the api server options
- if deployed as pod in kubeadm
  k describe pod -n kube-system
- can also see the manifest file at /etc/kubernetes/manifests/kube-apiserver.yaml
- can also see in /etc/systemd/system/kube-apiserver.service when deployed as Service
- also see with ps aux | grep apiserver

# Kube-Controller-Manager
It has many controllers like node controller, replication, deployment, job etc.

for example, node controller will watch for the status of nodes after every 5 secs if any node go down then wait for timeout period of 40s (grace period) etc. and it will mark the node down if not able to reach within 5 mins pod eviction period. if node is removed from cluster then it will move the pods to another node if part of replication. 

All this is done through kubeapiserver.

Same way, replication controller will  create another pod if one dies to ensure it is running desied replicas.

Same way as kubeapi-server, it can deployed as pod (kubeadm) or sepraretly. to check the options, see pod, ps aux or systemd service options.

notable options are controllers. Set the controllers as below if want to customize.
````
--controllers=
````

# Kube-Schedular
Note that only it is resposible for creating pod not deploying, which will be done by kubelet.

it will assign the pod to nodes based on criteria like cpu, memory requirement, taints or tolerarions or node affinity etc.

for example 2 nodes with cpu free 6 and 4. it will rank the node with 6 cpu as highest even if cpu for pod is required as 2 cpu.

You can also customize it and create your own scheduler also.

you can run it as service (as separate), pod (kubeadm). you can also see it through ps aux, pod, systemd service.

# Kubelet
it is like captain of the ship. point of contact for apiserver. 

request the cre to create container

It is always installed manually. it can't be done by kubeadm. use the ps aux to grep kubelet and see options

Note: In kubelet, there is a plugin called **cAdvisor** which reports the performance metrics using kubelet api. These metrics can be then used by metrics server(Heapster deprecated and trimmed to create lighter version called metrics server) to show statistics. 

# Kube-Proxy
Application communicate in k8s cluster using kube proxy. It will watch for any service creation and try to ensure any pod/application is reachable to the dest application pod using that service.

kube proxy is deployed on each node.

Oneway to do this is with iptables. It will create an entry in the iptables for service's ip mapping it to pod ip on that node.

As per my understanding, this could be possible because service is a virtual thing and just an exists as dns entry with ip (doesnt need to be dynamic like pod as it is not a container running) and which is mapped to the pods using end points. pods can't be used directly as their ip address is dynamic and can change after pod is restarted/crashed etc.

kubeadm deploy it as dameonset (pod)

or can be installed from github

# Kubectl apply
- Whenenver 'k apply' command is used to deploy resources in k8s cluster, it will always save the applied yaml as json object annotation in the live object. Annotation name is 'kubectl.kubernetes.io/last-applied-configuration'
- Suppose  after apply, you scale replicas to 2 using imperative command. it will not affect last applied configuration.
- Now suppose you again modify the yaml with image name, remove some label and use apply to do this. It will update the image, remov label, ignore the replicas change and leave it to 2 as set by imperative command. it will update the last applied configuration with the new yaml.

To see what changes will be applied, you can use the diff command.
````
k diff -f file.yaml
k apply -f file.yaml
````

to delete the objects, you can use delete file or apply with --prune
````
k delete -f file
k apply -f file --prune
````

to get the live object use 'k get -f file -o yaml'. This will work like 'k get po'

last applied config - this is also helpful to remove any field which are present in last applied but not present in live config, it will clear those fields from live object.

# Pods
## Multi container
Pods can run multiple containers in a single pod. all containers will share same fielsystem, volumes, network namespace, ip address, localhost etc.

## But why do we need this?
We break monolith applications into microservices. Each microserice does a specific job and a specific team can work on the service to make it agile. Sometimes, we might have resuable/common components that have same specific job to do but they are scattered across multiple microservices. For example, collecting jmx stats. This could be something required for multiple service and each have the duplicated code to get those stats.

Hence microservices can be broken further into application logic and reusable coomponents. application is in one container and the reusable component is in other container.There could be multiple/common patterns found by developers per their usecases.
Below are 3 common patterns identified for containers packaged into pods

### Sidecar
This is the pattern where containers share the filesystem. For example, one container writes into the filesystem and other read that from filesystem. 

**Application Container** -> **Filesystem** -> **Log Saving container**

### Ambassador
In this pattern, Application container uses a proxy to connect to outside world. For example, application container uses proxy to connect to redis cluster. application will always hit localhost. So that will act as redis server for application. but proxy container will segregate the read and writes and will send the reads to redis worker nodes and writes to redis master node.

**Application Container** -> **Proxy Ambassador container** -> **Redis Cluster (outside the pod)**

### Adaptor
Monitoring agent is an example of adaptor pattern. Application is running without any knowledge of adaptor container. Adaptor is monitoring the application and collecting stats.

**Application Container** <- **Monitoring Adaptor container** <- **Centralized Monitoring System (outside the pod)**

# Deployment
As per my understanding, we need deployment to make upgrading/updating application easy. We can ensure no of desired pods  using replica sets. With deployment with roll updates using many deployment strategies like blue-green, canary etc.

# Namespaces
- isolation of environments like dev, prod etc
- limit the resource for namespace using `ResourceQuota` etc.
- `k create -f file.yaml -n dev` to create resource in dev namespace if not specified in file.
- `k config set-context $(k config current-context) -n dev` will set the dev namespace in the current context and all queries will go to dev namespace.
