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
A pod can be scheuled on any worker node. So how to enable communication between containers/pods across the nodes? Kube proxy does this. it will define the rules etc to ensure this.

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
