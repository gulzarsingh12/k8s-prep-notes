# Security primitivies

There are multiple things we may need to do to secure a cluster. Some of them can be as below
* To **secure node**/vm - We can restrict the access to machine/nodes by setting up the ssh auth key and disabling password based authentication to login.
* To **secure the cluster**
  * **Authorization** - We can restict the access to the used using RBAC, ABAC, Node Authorizer, Webhook mode
  * **Authentication** - we can use certificates, LDAP etc for users. we use service accounts for applications.
* **Data in transit**
  We can use TLS certificates for all communications to encrypt the data in transit
* **Data at rest**
  we already see in etcd section how to configure the secret key in encyption config to ecrypt the data by kube apiserver while saving to etcd.
* Incoming/Outgoing access to pods
  We use **network policy** to restrict access between pods

# Authentication
There are multiple ways to set up authentication
* **Basic Auth**
  We can setup basic auth in kubeapiserver using cmd line option **--basic-auth-file**
  ````
  --basic-auth-file=/etc/kubenetes/auth/basic-auth-users.txt
  ````
  Note that this is deprecated in v1.19 and no more available and not recommended too

  format of file is as below:
  pasword,user,userid
  ````
  pwd1,user1,u0001
  ````
  you can also add group, in that case it will be
  pwd1,user1,u001,group01

  To call kubeapiserver
  `curl http://server/<api> -u user1:pwd1`
* **token based auth**
  We can setup token based auth in kubeapiserver using cmd line option **--token-auth-file**
  ````
  --token-auth-file=/etc/kubenetes/auth/token-auth-users.txt
  ````
  format of file is as below:
  token,user,userid
  ````
  rerfrr3derdf345r4,user1,u0001
  ````
  you can also add group, in that case it will be
  rerfrr3derdf345r4,user1,u001,group01

  To call kubeapiserver
  `curl http://server/<api>  --header "Authorization: Bearer rerfrr3derdf345r4"`

  This is not as secure way but good to understa d the basics. Also better to use volumes to provide the files to pods.

# Service Account
Since v1.22/1.24 service account has no more secret token defined due to security and other reason. It is recommended to use TokenRequestApi to generate the token. 

* In pods, to use api, token is mounted as project volume at /var/run/secrets/kubernetes.io/serviceaccount
* As secret is not created now so token can be generated with below
````
k create token <serviceaccount>
````
To see the jwt token
````
k create token default | jq -R 'split(".") | select(length>0) | .[0],.[1] | @base64d | fromjson'

{
  "alg": "RS256",
  "kid": "isXkD9tTaUM6kbSo1WkNGwl8hmZF2xT5T9LSVqFm-Ms"
}
{
  "aud": [
    "https://kubernetes.default.svc.cluster.local",
    "k3s"
  ],
  "exp": 1692859408,
  "iat": 1692855808,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "default",
      "uid": "b25eeeb9-c049-44cd-858e-234787b00cd4"
    }
  },
  "nbf": 1692855808,
  "sub": "system:serviceaccount:default:default"
}
````
It will look like below as projected volume in pod. In serviceAccountToken, there can be optional field `audience: api`. if not specified as in below, default is the identifier of api server.
This token is audience and time bound. if time not specified, it is 1 hr.
````
- name: kube-api-access-ph4r4
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
````
When mounted in pod, it will create below files.i.e. token, namespace, ca.crt
````
k exec -i nginx -- /bin/sh -c "ls -lrt /var/run/secrets/kubernetes.io/serviceaccount"
total 0
lrwxrwxrwx 1 root root 12 Aug 24 06:00 token -> ..data/token
lrwxrwxrwx 1 root root 16 Aug 24 06:00 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 13 Aug 24 06:00 ca.crt -> ..data/ca.crt
````
