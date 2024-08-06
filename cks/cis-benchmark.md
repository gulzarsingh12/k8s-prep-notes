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
    
