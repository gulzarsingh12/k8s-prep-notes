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
Set `anonumous-auth=false`, this will ensure that only authenticated user can connect to kubelet. there are 2 auth modes tls and bearer token. we will use cert here to discuss
  
when client-ca-file and cert and key are set for kubelet, it will enable the auth with cert. from client also, set the tls cert, key to authenticate. like by kubeaoi-server.

## authorization
set `authorization-mod=Webhook`. valid values are **AlwaysAllow** and **Webhook**. when set always allow is risky. set to webhook, then it will send it to apiserver to authorize it.

## disable readonly port
set `read-only-port=0` this will disable the 10255 port ofr read only access. if value is set 0, it means disabled

So in summary we can do authenticate, authorize and diable read port to secure kubelet. Set all the above as discuss in service or kubeletconfiguration removing - (dash) with caps.

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
