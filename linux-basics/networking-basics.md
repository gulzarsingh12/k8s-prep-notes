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
