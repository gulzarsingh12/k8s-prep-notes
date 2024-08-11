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
