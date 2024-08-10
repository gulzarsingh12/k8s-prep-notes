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


