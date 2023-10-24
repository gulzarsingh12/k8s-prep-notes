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

### Running as Pod (kubeadm)
- check if control plane components running. `k get po -n kube-system`
  if any component have any issue check the logs like `k logs kube-apiserver -n kube-system`
  check config files etc if confiured properly. Check certificates or their paths etc

### Running as service (init System)
- `systemctl status kubelet.service` or `service kubelet status`
   check config file path and entries in file.
   check configured service inside `/etc/systemd/system`
- check logs using `journalctl -u kubelet.service`

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

- Check if cni plugin is installed. Check `/etc/cni/network.d` `/opt/cni/bin`
- check cni

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
Check the 
