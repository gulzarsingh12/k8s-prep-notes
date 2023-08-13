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
