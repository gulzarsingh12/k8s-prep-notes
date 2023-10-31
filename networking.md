# Container network model (CNM)
To use kubernetes as orchestration, networking plugins has to follow a stabndard called CNI (Container network Interface). There are other orchetration like mesos, rkt etc also follow CNI. However, docker doesn't follow this standard and have defined its own called CNM (Container network model).

It doesn't mean docker can't be used with kubernetes. it is possible by creating docker first without network and then add the container to network.
Below is how kubernetes does it.
````
docker run --network=none nginx
bridge add **fefemfduw** /var/run/netns/**fefemfduw**
````

# CNI

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

# CNI

# Pod Networking
Below are the requirements for any network plugin to implement the networking for pods
- 
