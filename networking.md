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

#### To use check the port listening on node
````
netstat -plnt
````
