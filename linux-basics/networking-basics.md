# DNS

## etc/hosts
if there is internal network and we have few entries. we can specify those in /etc/hosts. `ping db`. here we can add `db` in /etc/hosts like `echo "db 192.168.1.3" >>/etc/hosts`

But what if there are many entries then we should have a dns server or we dont know this information yet and only available with system admin. we can have this dns server to give us this information.

## dns server
To connect to the dns server, we need to enter the dns server host ip in our host. `/etc/resolv.conf` is the place where it is defined.
it should be enter as `nameserver 8.8.8.8`.

### /etc/resolv.conf
````
nameserver  8.8.8.8
````
Every time we look for the name resolution, system will look for the nameserver as in above is 8.8.8.8 and will ask for the A record for the dns name we provided.

Suppose, we have our internal dns server which is serving the records for our internal systems and we try to look for `ping www.facebook.com`. it will result in **name resolution failure** error. To resolve this we can add the server like below
````
nameserver 192.168.1.2
nameserver 8.8.8.8
````
With the above it will now look up for internal dns server and then google dns server and it will work and it will resolve www.facebook.com. However, we have to add it to every machine in our network..

Better solution is to add below to our dns server `192.168.1.2` above as `Forward All to 8.8.8.8` This we dont need to add 8.8.8.8 in each machine in our network and our internal dbns server will forward the request to google dns server which we dont have entry for.

## dns name resolution order
Usually it will look for any host name in `/etc/hosts` and then go to dns server. but this order can be changed. it is defined in the `/etc/nsswitch.conf` file. we can ask to resolve from dns server before `/etc/hosts` by changing the order.

### /etc/nsswitch.conf
````
hosts:    files dns
````

## domain search
whenever we search for any dns name in our host, it will look into for that name in dns server where. for e.g. web. it will look for web which might not be resolved. To resolve this in company network, you might have to provide the domain name.e.g. maps.google.com. so if you search for maps and search domain has google.com in it. it will resolve easily.

To do that we need to define like this
````
nameserver 8.8.8.8
search mycompany.com
````

In the above case, if I search for `web` name, it will look into dns server. if not found it will try web.mycompany.com to find it. 

Dns server can cache the entry to avoid missing hits. ttl is the time to live in cache.

## Record Types
There are many record types but below are some to explain for this usecase.

### A
A record is for ip address ipv4

### AAAA
AAAA is for ip address ipv6

### CNAME
This is synonym names for the server.e.g. food.web-server will have eat.web-server or hungry.web-server etc.

## tools
there are many tools. some of them are below:

### nslookup
This is the also used to resolve the ip address of the host from dns server. 
**Note** it will not check **/etc/hosts** entry but dns server directly.

### dig
This is better tool then nslookup and give more information. it will give more information.

# Network Interfaces

## Switch

How to connect 2 devices with each other? We can use switch for this. 

A switch is used to connect multiple devices with each other. There 2 types of switches, layer 2 (OSI layer 2, mac address) and layer 3 (osi layer 3, network layer). layer 2 switch use mac address and layer 3 switch use ip address. Some switch can do both. But most switches are layer 2 switches.
This is usually used to create local network (LAN)

### How it works
Suppose A want to connect to B. It will connect to switch. Support A is connected on port 1. B is connected on port 2. Switch has its CAM memory to remember this as below table.

MAC|Port
---|----
A|1
B|2
C|3

A ---  Switch --- B

If a data packet comes for A, it will check in the table for the mac and forward to the port 1. This happen at layer 2 (switch). so mac addresses are used. when packet come (layer 3), its header will have ip address.


#### local
If we take example above, When A want to connect to B using switch, it will try to resolve via mac address. switch doesnt need ipaddress. it will just check the mac address in its CAM memory table.

To list the interfaces on both A and B run `ip link` command. You should see the interface name like `eth0`

Then create the ipaddress on both the interfaces using 
A -> `ip addr add 192.168.1.2/24 dev eth0`
B -> `ip addr add 192.168.1.3/24 dev eth0`

Now if you ping B from A, it will work in case of switch as switch will work and connect both devices at layer 2 using mac addresses.

## Router
In case of switch we can connect with devices easily at layer 2. As it is layer 2 so it has to be connected physically using the interface to the port on switch. without which it wont work because its layer 2, hence no ip address.

But what about if A or B want to connect to C or D which are on some other network. Then Router is the way to connect in that case.

A/B --> Router <---C/D

You can check the routes in routing table using `ip route` or `route` command.
It will show destination, gateway used to connect.
Suppose C has 192.168.2.2 as ip address, then route has to be added as below on A/B to connect to C/D

`ip route add 192.168.2.0/24 via 192.168.1.1`

This way A/B can connect with C or D. To c to connect to A/B, we need similar entry on c as `ip route add 192.168.1.0/24 via 192.168.2.1`

Here 192.168.1.1 and 192.168.2.1 are ip addresses on the router.

## Gateway
Gateway is the router which acts as exit and entry point for a network. for example as router in home wifi netowork also acts as gatway to internet.
