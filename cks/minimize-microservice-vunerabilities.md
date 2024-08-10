# Pod Security Policy

- PSP's are deprecated in v1.21 and removed in v1.25. However they can be asked in exam or in real world to upgrade existing 
clusters from PSP to its replacement PSS/PSA.
- PSP are not enabled by default and need to explicitly enable as admission controller. `--enable-admission-plugins=PodSecurityPolicy`
- after enabled, we need to create PSP
  ````
  apiVersion: policy/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: example-psp
  spec:
    previleged: false
    runAsUser:
      rule: MustRunAsNonRoot
    seLinux:
      rule: RunAsAny
    requiredDropCapabilities:
     - CAP_SYS_BOOT
    defaultAddCapabilities:
     - CAP_SYS_TIME
    volumes:
     - persistentVolumeClaim
  ````
- if you see above, it has requiredAddCapabilities, which means its not only validating admission controller but mutating admission controller too. this is not applicable in PSA/PSS
- Now after adding the above PSP, if you try to create a Pod, it will reject the request.
- to actually make it working, need the authorize the pod to use the PSP. this is done via adding a role with `verb: use, resource: PodSecurityPolicy, resourceName: example-psp` and bind it to the serviceAccountName for the pod. which is default by default. so binding needs to be defined for binding the role to default sa.
- after this much hassle, you can see the pod being as PSP controller will approve the request.
- So it is not easy to configure it and need below
  - enable it in api server as admission plugins while PSA is enabled by default
  - create policy
  - create appropriate role and rolebinding
  - increase complexity
- this was the reason of removal for this, its not easy to use and can fail other deployments and services where not applicable
- also its not easy to find where it is applied and where not.

# PSA
