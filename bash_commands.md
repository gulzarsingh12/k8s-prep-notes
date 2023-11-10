# K8s commands to remember before exam
- To check the iptables entry in kube-proxy if exists `iptables -L -t nat | grep db-service`
- check logs or configmap to check kube-proxy for firewall rules using which iptables, ipvs etc
- service cidr is in kube-apiserver args `--service-cluster-ip-range` and controller manager
- pod cidr in `--pod-network-cidr` in kubeadm init 

# Vi Commands
- y to copy, p to paste, d to delete
- v to mark, arrow to select and move up down, > to ident right, < to ident left, . to continue

# General
- To apend logs to a file. this will append instead of overriding `k logs pod >>pod_error.log`

# Systemd

## to check init system
`ps -p 1`
output contains  systemd if init system is systemd.

# for loop

## iterate over range 1... 10
`for i in $(seq 1 10); do echo $i; done;`
prints 1 2 3 4 5 6 ... in each line

## iterate over provided value in condition
`for i in hello world 1 10; do echo $i; done;`
prints `hello world 1 10` in each line

# if

## if only
`status="true" && if "$status" == "true"; then echo status is set; fi`

## if else
`status="false" && if "$status" == "true"; then echo status is set; else echo status is not set; fi`

### if else if
`type="color" && if [ "$type" == "color" ]; then echo orange; elif [ "$type" == "hero" ]; then echo spiderman; else echo type not known; fi`


# Install K8s
- Follow k8s documentation
- Install prerequisites
- Follow install kubeadm on k8s docmentation
- Create cluster. add advertise ip, extra sans, pod cidr based on network plugin
- add pod network, check urls . ensure interface name set for flannel in kube-flannel.yml
- check nodes status
