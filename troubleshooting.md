# Logs
- Logs path can `/var/log/pods` or `/var/log/containers`
- Logs for kubelet `/var/log/syslog` or `journalctl`

# Commands
- to get cluster info `k cluster-info`
- to know component status `k get componentstatues`

# Application Failure

### Service
- Check endpoints of service
- Check labels match with pod

### Pod
- Status running? `k get po`
- restart counts  `k get po`
- check logs `k logs <pod-name>`
- use `--previous` if pod is down
- to tail the logs use `-f`
- check the pod related stuff like events, volumes etc using `k describe po`

# Control Plane Failure
- Check node status `k get nodes` if not ready try describe to see the issue
- Check pods running `k get po` if not running try describe to see the issue
- Follow below sequence to find the issue with controlplane
   1. Check `k logs <pod>`, this can work for other than apiserver pods created by kubeadm. So first thing is to check if pods are created or started as service. if service then check `journalctl` otherwise `k logs`. Hence this step wont work for `kube-apiserver` if using kubeadm setup.
   2. Try checking logs `crictl ps -a` and find the container id for `kube-apiserver`. Check logs at `/var/log/pods` or `/var/log/containers`.
   3. Somnetimes problem in manifest or something else wrong. Try checking the kubelet logs. `/var/log/syslog` or `journalctl`
      ````
      tail -f /var/log/syslog | grep apiserver
      or  journalctl | grep apiserver
      ````


### Running as Pod (kubeadm)
- check if control plane components running. `k get po -n kube-system`
  if any component have any issue check the logs like `k logs kube-apiserver -n kube-system`
  check config files etc if confiured properly. Check certificates or their paths etc

### Running as service (init System)
- `systemctl status kubelet.service` or `service kubelet status`
   check config file path and entries in file.
   check configured service inside `/etc/systemd/system`
- check logs using `journalctl -u kubelet.service`
  To tail 10 lines of log `journalctl -u kubelet -n 10`
  To continue logs, pass `-f`

# Worker Node
- Check node status `k get nodes` if not ready try describe to see the issue
- check disk pressure, memory pressure, pid pressure etc.
- Mostly status is unknown if kubelet is not able to send status to kube-apiserver
- if kubelet is crashed, bring it up
- check cpu and memory using `top` and disk space using `df -h`
- check service status `service kubelet status` or `systemctl status kubelet`
- check logs `journalctl -u kubelet`
- check certificates `openssl x509 -in <path> -text`

# Networking

- Check if cni plugin is installed. Check `/etc/cni/net.d` for cni conf and `/opt/cni/bin` for cni bin dir.
- Check cni network plugin, cni bin, cni conf dir in `ps -aux | grep kubelet`
- conf file will have name which should be present in cni/bin.
- if multiple conf files, then it will read in ls lrt order.
- Check if pods are running. for example if its weavenet, `k get po -n kube-system`, check if pods with weave are running.

## Download

### Weavenet
````
wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
````

### Flannel
````
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
````

### Calico
````
wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/calico.yaml
````

## CoreDns
- Check the core-dns pods are running
- check the kube-dns service in running. check endpoints etc. Check config file etc
- check logs for container or systemctl etc.
- Check configmap is mounted as volume for config file etc

## Kube-proxy
- Check if kube proxy deamon set is running. `k get ds -n kube-system`.
- Check configmap is mounted as volume for config file etc
- check kube-proxy is running inside container `netstat -plan`

