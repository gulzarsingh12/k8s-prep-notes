# Benchmarks

## CIS
Center for Internet Security. It provides th best practices to follow for various os, kubernetes, clouds etc. 

One way is to register to the cis and download the pdf with best practices and go to each eaqch section verify against your installation.

Another way is to run the tool provided by CIS. **CAS CAT Lite** or **CIS-CAT Pro**

### Ubuntu
Run with beloe command using **CIS-CAT-Pro**
````
./Assessor-CLI.sh -i -rd /var/www/html/ -nts -rp index
````
- i - interactive
- nts - no timestamp
- rp - report file name. default format is html

Please note that CAT Lite doesnt support kubernetes and provides limited features. CAT Pro can be used for K8s.

## KubeBench
Kube-bench from aqua security.
This tool can be run for running benchmark tests on kubernetes cluster.

Follow the instructions for download and run.. like below to pass config file.
````
./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml
````
### flags
`kube-bench run --targets=master,node,etcd,policies --check="1.1.1,1.2.1" --skip="1.3.1,1.4.4"`

# Security Primitives
Below are security primitives

## Secure hosts

- Disble password based authentication on hosts
- Enable ssh authentication
- Disable root access

## Secure Kubernetes
- controll acess to api server
  - authentication
  - authorization
    - rbac
  - all controlplane components communication can be secured using tls.
  - all pods/application communication can be secured using network policies


# TLS
## Generate in single command
I have seen this example in killerkode.
````
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.key -out cert.crt -subj "/CN=world.universe.mine/O=world.universe.mine"
````

## CA
Generate ca private key and then cert for ca 
- generate private key `openssl genrsa -out ca.key 2048`
- generate csr `openssl req -new -key ca.key -subj "/CN=kuebenetes-ca" -out ca.csr`
- generate the cert for ca `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`

## client
### kube-admin
User to login to cluster as admin

Generate for kube admin
- generate private key `openssl genrsa -out admin.key 2048`
- generate csr `openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr`
- generate the cert for ca `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`

You can also pass these to rest api call to kube-apiserver or set in kube config under certificiate-authority or certificate-authority-data

### control plane
Same process as client but name should be prefixed with system. `-subj "/CN=system:kube-scheduler"`

Same way generate certs for kube-controller-manager, kube-proxy etc but remember the name prefix as `system:`

For **etcd**, there can be multi certs to support HA. those should be gone to peer-cert-file and peer-key-files as comma separated entries.

For **apiserver** there can be multiple names
so it can set to openssl.cnf file and it can be passed to openssl command as `-extfile openssl.cnf`

For **kubelet**, there can be many nodes. 
for server side certs, each node sert should be set into **KubeletConfiguration** (kubeket-config.yaml)
for client cert(this is required by each kubelet to connect with apiserver as client). Please rememeber to set group as `/O=system:nodes` and name as `/CN=system:node:node01` etc.

### cipher suites
to make it more secure, you can specify the cipher suites to use.
- kube api server `--tls-min-version=VersionTLS12 --tls-cpher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`. default version is `VersionTLS10'. max version is `VersionTLS13`
- etc `--cpher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`

## View Cert
To view the cert `openssl x509 -in admin.crt -noout -text`

## CSR
- Generate the csr and set it to **CertificateSigningRequest** yaml. base encode using `base64 -w 0` and do k apply. view the csr `k get csr`
- after that approve the csr `k certificate approve jane`
- after approval get the generated cert to share with end user. `k get csr jane -o yaml`. decode the base64.

# Kubelet Security
There are 2 parts mainly to secure in kubelet.i.e. authentication and authorization
kubelet exposes 10250 and 10250 ports to access its api. 10250 is to provide api to access kubelet services but 10255 as a read only to access metrics of the pods and nodes etc. This is risky and should be disabled.

## auth
Set `anonumous-auth=false`, this will ensure that only authenticated user can connect to kubelet. there are 2 auth modes tls cert(https) and bearer token. we will use cert here to discuss
  
when **client-ca-file** and cert and key are set for kubelet, it will enable the auth with cert. from client also, set the tls cert, key to authenticate. like by kubeapi-server.

## authorization
set `authorization-mod=Webhook`. valid values are **AlwaysAllow** and **Webhook**. when set always allow is risky. set to webhook, then it will send it to apiserver to authorize it.

## disable readonly port
set `read-only-port=0` this will disable the 10255 port ofr read only access. if value is set 0, it means disabled

So in summary we can do authenticate, authorize and diable read port to secure kubelet. Set all the above as discuss in service or kubeletconfiguration removing - (dash) with caps.

## NodeRestriction
This is enabled in most of the kube adm deployment but its not by default hence need to do at `--enable-admission-plugins` in **apiserver**. This will restrict the access of a kubelet to only create labels etc on the node and pods created locally. Also it doesn't allow it to create labels with `node-restriction.kubernetes.io` on locally and any label on other nodes.

# Port Forward
- To access kube api server `kubectl proxy` will start on localhost using port 8001. this way it will not need to pass tls cert,key,ca with every request to api server.
- But what about the application? to access a service in this case, use port forward. `kubectl port-forward service/web-service 28080:80`. then you can access the service on local as `curl http://localhost:28080`. 
Note that port-forward works for pods, services, deployments etc etc..

# Kubernetes dashboard
- Do not expose kubernetes dashboard to outside world.
- if required, try to use it locally via kubectl proxy etc.
- or use auth proxy to authenticate user first and then redirect to dashboard
- dont expose directly via load balancer etc.
- 2 ways to use, token or kubeconfig. to use token `create sa my-user`, `k create token my-user` better expiry for the token. set the role and role binding for the user with least permissions.

# Verify Binary
- for linux use `sha512sum kuber.tar.gz`
- on mac `shasum -a 512 kuber.tar.gz`

# Secure Docker
Docker run with unix socket on the linux os. It is easy to use it locally as docker cli uses socket to connect with docker daemon and secure as only acceisble locally.

But when it needs to connect with other host, it means if cli is run on another host then docker server should expose it on http(s).
To secure the traffic between client and server in this case, set below
`dockerd --host=tcp://192.168.1.10:2376 --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem`

there are 2 places to specify this, either in service unit file or in `/etc/docker/daemon.json`.
````
{
  "hosts": ["tcp://192.168.1.10:2376"],
  "tls": "true",
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem"
}
````
if any property specified at both places and have different value, it will throw error.

with this data in transit will be encrypted but still anyone can access it. to ensure only authenticated user can access, enable cert based authentication using tlsverify.
`dockerd --host=tcp://192.168.1.10:2376 --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem --tlsverify=true --tlscacert=/var/docker/caserver.pem`

This way above will only allow acces with client certificate. So for client side, when they access, they need to get the **clientcert.pem** and **clientkey.pem** **generated** from **caserver.pem**. this way only authenticated user will be able to access the system.

With insecure docker, someone can
- delete volumes/data
- delete containers
- gain access to host
- run container for bit coin mining etc.
  
https://sysdig.com/blog/7-docker-security-vulnerabilities/ 

## PID namespace
`docker run --name app1 -d --pid=private nginx:alpine sleep 1d`
- `--pid=private` start docker container in its own pid namespace.this is default also. other modes are
  - `--pid=host` will share the pid namespace of host and will have access to host
  - `--pid=container:<containerId>` will share the pid namespace of other container. like `--pid=container:app1` will show the processes running in container app2 in app1 and vice versa.
-  `-d` run in detached mode
-  `sleep 1d` is the command pass to container to run

`docker exec -it app1 /bin/sh` will attached to the bash terminal of running container.  

# Network Policy
To test connectivity `k run tmp --image=nginx:alpine --rm -i --restart=Never -- nc -v 1.1.1.1 53` 
 `nc -v host port` is better to test anv availabel easily

# RBAC
- Ensure api server is set to use RBAC `--authorization-mode=Node,RBAC`
- ensure `--insecure-port` is set to 0.
- ensure `--insecure-bind-address` **is not set**. In case its required, it could be set to localhost. it shouldn't be to any network reachable ip address.
- `--anonymous-auth` to false
