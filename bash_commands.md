# K8s commands to remember before exam
- To check the iptables entry in kube-proxy if exists `iptables -L -t nat | grep db-service`
- check logs or configmap to check kube-proxy mode.i.e. iptables, ipvs etc
- service cidr is in kube-apiserver and controller manager args `--service-cluster-ip-range`
- pod cidr in `--pod-network-cidr` in kubeadm init.
- To fetch a page in busybox `wget -O (filename|-) [url]`  - means stdout e.g. `wget -O- https://stackoverflow.com` to write the file to std output. Or `wget -O index.html https://stackoverflow.com` or `wget https://stackoverflow.com` also download the index.html at current path. To set timeout `wget -T 2 -O- https://stackoverflow.com` will downloadand print output to stdout with timeout 2 secs.
- To fetch a page using curl. nginx has curl but busybox has wget. `curl https://stackoverflow.com` or `curl -o index.html https://stackoverflow.com`

# Vi Commands
- y to copy, p to paste, d to delete
- v to mark, arrow to select and move up down, > to ident right, < to ident left, . to continue
- to go to top of the file `gg`, to end of file `shift + g`
- To search `/hello` and then `n` for next occurence or enter to stop and search backward `?hello`
- To search and replace all occurence `:%s/hello/hola/g` to replace
- To search and replace first occurence in current line `:s/hello/hola/` to replace . `%` mean all line in file to search and `g` means all occuerence to replace in current (`s`) line or all lines (`%s`)
- to delete all occurences in file `%s/hello//g`
- vi ~/.vimrc
  ````
  set et
  set ts=2
  set sw=2
  set sts=2
  ````
- vi ~/.bashrc
  ````
  alias kc="k create -f"
  alias ka="k apply -f"
  alias kr="k replace -f"
  alias kd="k describe"
  alias kgp="k get po"
  alias kga="k get all"
  alias kdp="k delete po"
  alias kg="k get"
  alias kgn="k get nodes"
  alias kgs="k get componentstatuses"
  export do="--dry-run=client -o yaml"
  export now="--force --grace-period=0"
  nk="-n kube-system"
  ````

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
