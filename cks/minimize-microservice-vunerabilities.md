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
- easy to use
- safe
- enabled by default `PodSecurity`
- it is applied via adding a label to namespace. `pod-security.kubernetes.io/<mode>=<securitystandard>`
- There are no additional config is like config for PSP created eariler.
- Just defined label as namespace level as above.
- There are 3 modes and 3 standard

  PSA
  - enforce
  - audit
  - warn
 
  PSS
  - previleged
  - baseline
  - restricted
 
- when **enforce=restricted**, it is very much restricted and reject any violating request
- **warn=restricted**, it will just get warning and still allow request if violated
- for example
  - for namespce payroll, **enforce=restricted**
  - for namedpace hr,  **enforce=baseline**
  - for namespace dev, **warn=restrcited**
  - 
## admission configuration
to change what is applied to the kubernetes as PSA is done by changing the admission configuration. this is applied as `--admission-control-config-file` in kube-apiserver.

# OPA
- It is like validating some input for a web application. for example, if application get a request from web, application want to reject the request if user != john. 
-  Same above thing can be deployed on kubernetes object when created.e.g. pods, services etc.
-  Opa can be install and run as server, from application, request can be forwarded to opa server to validate. opa file is writen in reg language
````
package httpapi.authz

import input

default allow = false

allow {
  input.path = home
  input.user = john
}
````
- once policy is avaiable as above , this should be uploaded to opa server at /v1/policies/<policyname> `curl -X PUT --data-binary @sample.rego http://localhost:8181/v1/policies/samplepolicy`
- once policy uploaded, then call the server from app to validate
- to test policy file `./opa test example.rego`

## opa gatekeeper
`kube-mgmt` manages policies / data of Open Policy Agent instances in Kubernetes. so opa server's policies are managed by this kube-mgmt. to use it, create configmap from the policy rego file. it will automatically load into opa.

this is new approach as compare to above and better.
- install opa gatekeeper as container in k8s

### opa constraint framework
it has 3 things
- what - requirement
- where - k8s admission controller
- how - what to check and what action to take

example  (what - expensive namespace has billing label, where - target is admission.k8s.gatekeeper.sh, how - check the label if missing then reject.

it has **ConstraintTemplate** to specify what to check. apply `k apply -f constrainttemplate.yaml`
````
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
````

````
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["gatekeeper"]
````

# Container Sandboxing
Container run directly with syscall to kernel which is on host. hence it is insecure due to other multiple containers can be affected and even host itself. someone can esclate the previleges and control the host. This is not like vm where everything is dedicated to per vm.

## runtime classes
gvisor and kata use different runtimes to run the containers. gvisor uses **runsc** and kata uses **kata-runtime**. 

as both are oci compliant, we can use docker cli to run the containers.

`docker run --runtime kata -d nginx`
`docker run --runtime runsc -d nginx`

### kubernetes
We have to create first **RuntimeClass** first
````
apiVersion: node.k8s.io/vabeta1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
````
use `kata-runtime` for kata containers

In pod spec, provide `runtimeClassName: gvisor`

check process `pgrep -a nginx`, it wont show anything, that means it is sandboxed. we can also see if runsc is running. `pgrep -a runsc`

## gVisor
it allow additional layer between container and kernel. it will recieve the syscalls but its not directly to host.

Container
   |
Syscalls
   |
Sentry | Gofer (gvisor)
   | 
Linux kernel
   |
Hardware

### gvisor sandbox
contains 2 components
- sentry - independent application level kernel. it is designed for container in mind. so it supprts only fewer syscalls to avoid the flaws.
For example if app needs acces to file, it wont allow it directly access system files rather uses gofer to call to kernel to provide what is requested. it acts as middleman.
for network, it will use its own network stack to isolate it from kernel.

Every container will have dedicated gvisor to isolate. so it wont affect all other containers

However, disadvanatage 
- is that it may not work with every app
- due to middleman, more instrcutions to execute

## kata containers
as we know traditionally we use vm's which is more restricted as compare to containers. 

kata will use the vm approch. it will run every container in separate light weight vm. 

 - vm are lightwieght and performance is optimized but still some overhead of memory and space
 - another problem is that it needs bare metal because it runs a vm hence can't run vm inside vm. most of provider doesnt support. google cloud provide but needs to enable it. its not enabled by default.performance is very poor
 - on bare metal, it is fine to run and wont see performance issues like nested vm.


# mTLS

istio and linkerd can be used to do mlts. it is done at service level.

istio does by sidecar, it runs a sidecar to support encryption on each pod. so if podA is running app and istio sidecar, then sidecar will encrypt and send to podB's sidecar and which will then send to app in podB.

but this is done on case by case. so if some external service to podA then it wont do mtls but if podA to podB then it will.

- so podA - podB is called strict/enforced mode
- but podA -external svc is called permissive/oppurtunistic mode.

# Image Security
-  **modular** - dont build everytihng into single image. like db and web etc.. only one function in one image
-  **persist** -  dont store data in image rather use volumes or cache like redis
-  choose base/parent image which is safe/secure and specific to your need.
   - if need httpd, then use httpd image only
   - use official image so that its verified
- **uptodate** - image which is updated freqently. this means time to time updates to fix bugs and vunerabilties by maintainer
- use **distroless** images like google's gcr.io/distroless/*, this will remove extra uneeded package from image
- slim/minimal - only have minimal package not everything.. remove shells, packlage managers etc.
- for dev its fine, for prod - use lean images
- scan image for vunerability `trivy image httpd`

## private repo
you can use private repo to fetch from which is maintained by ur org and have approved and scanned images.
kubernetes allow secrets to save the cred's for docker under **imagePullSecrets**

## image policy webhook
there are couple of ways to whitelist the registry to fetch the docker images
-  one could be to write your own admission controller and whitelist the request
-  another could be using opa gatekeeper which we discussed already earlier to validate the name of docker image containing registry
-  third way is to use the built in controller from k8s. **ImagePolicyWebhook** with this you can configure the webhook server to serve the request.
  this plugin can be enabled in kube-apiserver in enable admission plugins. configuration to connect with server can be specified in AdmissionConfiguration via --admission-control-config-file.

Under the configuration, **defaultAllow=true** means that if server is not available then it will allow the request without validating it. it uses kubeconfig file format to configure the webhook server.

## kuebsec
this is tool to static analysis of kubernetes resource.

you can `kubesec scan pod.yaml` or `curl -sSX --data-binary @"pod.yaml" https://v2.kubesec.io/scan`

## trivy
this is to scan the docker images. this is from aquasecurity
CVS score >7 is high and can be high risk.

`trivy image nginx:1.18.0`

you can pass severity level. `trivy image --severity CRITICAL,HIGH nginx 1.18.0`.  it will list only high and critical severity CVE's

`trivy image --ignore-unfixed nginx:1.18.0` will report the CVE's which can be immediately fixed by upgrading the image.

to scan image as archive file 
````
docker save nginx:1.18.0 nginx.tar
trivy image --input nginx.tar
````
### best practices
- peridically rescan images as new vunerabilities are discovered over the time.
- can use admission controllers to scan images before pod is deployed.
- use private repo with prescan images
- integrate scan into cicd pipeline.
