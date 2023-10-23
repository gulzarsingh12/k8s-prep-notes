# Application Failure

## Service
- Check endpoints of service
- Check labels match with pod

## Pod
- Status running? `k get po`
- restart counts  `k get po`
- check logs `k logs <pod-name>`
- use `--previous` if pod is down
- to tail the logs use `-f`
- check the pod related stuff like events, volumes etc using `k describe po`

# Control plane
## Node
