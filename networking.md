# Container network model (CNM)
To use kubernetes as orchestration, networking plugins has to follow a stabndard called CNI (Container network Interface). There are other orchetration like mesos, rkt etc also follow CNI. However, docker doesn't follow this standard and have defined its own called CNM (Container network model).

It doesn't mean docker can't be used with kubernetes. it is possible by creating docker first without network and then add the container to network.
Below is how kubernetes does it.
````
docker run --network=none nginx
bridge add **fefemfduw** /var/run/netns/**fefemfduw**
````

# CNI

# Kubelet CNI Config
- In kubelet configuration check cni config at `--cni-conf-dir=/etc/cni/net.d` and cni binary at `--cni-bin-dir=/opt/cni/bin`
- cni bin directory have all the supported plugins.

#### To show the network type. For example to show the interface for bridge network
````
ip addr show type bridge
ip link show type bridge
````

#### To see the interface details with name
````
ip link show eth0
ip addr show eth0
````

#### netstat
`netstat -plant`
- -n  to show the ip addresses and dont show the dns name like localhost instead of 127.0.0.1
- -p  to show the program name or pid. like kube-scheduler etc.
- -l  to show the listening ports
- -a  to show all the ports connected or not
- -t  to show only tcp ports


#### To use check the port listening on node
````
netstat -plnt
````

### Deploy cni plugins

#### Weave net
wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
k apply -f weave-daemonset-k8s.yaml

#### Flannel
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
k apply -f kube-flannel.yml

#### Remove \n from text
````
tr -d '\n'
````

## Pod Networking
- Pod ip range can be found from the network plugin logs by checking ip alloc range. `k logs <weavenet pods> weave -n kube-system`
- I personally also checked the same for weavenet using `k get po <pod> -o yaml` and found IPALLOC_RANGE in yaml file but this may be true for weavenet only. hence not sure.

## Service Networking
- There is no actual process running like pods for services. service is just virtual concept and allocate an ip. Kube proxy watches the kube-apiserver to create service ip in iptables and add endpoints ips as target.  Service is cluster wide hence exists on all nodes in kubeproxy.
- service cluster ip range is defined in kube-apiserver. `--service-cluster-ip-range=10.96.0.0/12`.
- However pod have different ip range. for example 10.24.0.0/16 in this case. Both ranges should never overlap
- `iptables -L -t nat | grep db-service` will rule configured for the db-service ip. For example all traffic for 10.103.132.104 (service ip) will go to 10.244.1.2:3306 (pod). In case of nodeport, port is also configured for service. 
- Whenever a service is added then entry for the corresponding ip of the service for pods is added into iptables of the kube proxy
- the same addition of entry can be seen in logs. `/var/log/kube-proxy.log` Location of the logs may differ and this log sttement also depends on the verbosity of the log.
- use `ipcalc` utility to find the ip address range for given ip range. example `ipcalc -b 192.6.31.6`
