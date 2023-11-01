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
- In kubelet configuration check cni config at `--cni-conf-dir=/etc/cni/net.d` and cni binary at `--cni-bin-dir/etc/cni/bin`
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
