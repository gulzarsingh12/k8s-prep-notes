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

# TLS Basics
When we transfer data over network, we want to ensure nobody else can understand. we can scramble/encrypt the data. 

But how? There are plenty of cipher suites available for that. SSL cipher suites are considered insecure these days so not a recommended way to encrypt data. **SSL** v1 to v3 are available. As these are considered insecure, TLS is the current way to encrypt data. TLS v1 and v1.2 is considered less secure. In most browsers, **TLS** v1.3 cipher suites are used.

To encrypt data, we need a secret/encryption key. There are 2 types of encyption keys available.i.e. symmetric key, asymmetric key. A **symmetric** key is an encyption key used in encyption algorithms when encyption and decryption of data is done using the **same key**. An **asymmetric** key is **public/private** key encryption. Public key is used to encrypt the data and private key is used to decrypt the data.  However, it is possible to use either key to encrypt data and other key to decrypt data in case of **RSA**. It means, if you encrypt data with public key, you can decrypt data with private key but vice versa is true. Please note that not all the algorithm support that.

## Certificate Validation
There are 3 types of validation for certificates

### Extended Validation certificates (EV)
This is the most stringent validation process. It has highest level of encryption, validation and trust. It includes business physical address, proper certificate application and exclusive right to use the domain.

This is mostly used for protecting sensitive data such as financial transactions, medical records etc.

### Organization Validation certificates (OV)
This is the less stringent validation process than EV. Applicants must prove ownership to the domain.

commerical business use these certificates to build the trust among customers.

### Domain Validation certificates (DV)
This is the least stringent validation process among three. Applicants can prove ownership by responding to email or phone call.

It doesn't provide complete information about applicant's organization/business. It does not provide high assurance to users hence it is more suitable for usecases like blogs etc.

## Certificate domains
There are 3 types of domains of certtificates

### Single domain
These are certificates to protect only one domain or subdomain. For example, https://example.com. User can't use it even for subdomain like http://blog.example.com.

### Wildcard
It will protect domain and as well as sub domains. For example, http://example.com, http://blog.example.com, http://shop.example.com etc.

### Multi-domain
It will protect multiple domains. For example, http://example1.com, http://domain2.co.uk, http://shop.business3.com etc.

## How TLS works
I am explaining below TLS end to end with steps below:

### Process

#### Handshake Phase
##### Client Hello
Client/browser will send Hello message to server which includes the information about supported cipher suites and a random value.
##### Server Hello
Server will respond to hello message and selecting the cipher suite etc. and a random value

##### Key exchange
Server will send its public key certificate other key key exchange information. Server may share additonal exchange information like PSK (PreSharedKey)

##### Key Derivation
Using the information exchaged between client and server and random values from both sides, a shared secret is derived.

#### Session Key Generation
client will generate a symmteric session key from the shared secret. a KDF(Key derivatiob function) is used to derive the session key, initial vector (IV) from shared secret.

Using RSA or ECDH (Diffie Hellman) key exchange mechanism, client will encrypt the session key using RSA/ECDH assymetric key (public key) and send to the server to be used for future communication.

#### Data Exchange
Once session key is shared with server, client will use the session key to ecnypt data to send to server and which server can decypt.

#### Session Termination
Once the session is terminated, session key is no longer used, future session will initate the same process above for handshake and establish new session.

### Process In detail
When public key certifficate is recieved, it will be verified. To get the verifiable certificate, it has to be come from CA.

Server will generate a private key using any ssl tool. for example openssl. Using **openssl**, rsa private key is generated.
Using private key, a **CSR** (Certificate signing request) is created and send to CA. CA will create a public key certificate. It will contain **public key, digitial singature** signed by CA using CA's private key and other informtion like **CN**, **SAN** etc.

Once the public key certificate is received by client, it will read the certitficate and get **digital dignature** signed by **CA** using its **private** key. It will **validate** the **signature** using CA **public** key. It is possible that the CA used by server certificate might be an **intermediate** CA which has further CA in the hierarchy. client will **continue** the **validation** across the chain till its reaches the **root** Certificate. Once it will reach one of the **root CA stored in browser**/operating implicitly. if any of the CA validation **failed**, it will fail the validation check. Root CA are **publically available already on all the browser** or opeerating system so it is not requested during the validation from web.

Once the validation is completed then session key is exchanged with server. Server send the **acknowledgment** back encrypted using the session key. if client is able to decrypt acknowldgement and validate then handshake is completed and future communication will happen using the session key. 

RSA assymetric public key is not required anymore for the session as session key is securily exchanged. 

Why can't we use **RSA** key for encrypting subsequent requests? Because it is **slow**. **symmetric** key is **faster** for encryption and decryption. hence once session is established, **assymtric** key is not required for that session.

This whole process of cryptgraphy to secure the user data is called **PKI (Public key infrastructure)**

# Certificate Creation
To create a certificate, following steps are required. We are using openssl.

## CA
**private key**
````
openssl genrsa -out ca.key 2048
````
**csr**
````
openssl req -new -key ca.key -out ca.csr -subj "/CN=KUBERNETES-CA"
````
**CA cert**
````
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
````

## Client
**private key**
````
openssl genrsa -out admin.key 2048
````
**csr**
````
openssl req -new -key admin.key -out admin.csr -subj "/CN=kube-admin/O=system:masters"
````
**public key cert**
````
openssl x509 -req -in admin.csr -CAKey ca.key -CA ca.crt -out admin.crt
````

Please note that CN name for control plane components like kube-scheduler, kube-controller-manager, kube-proxy should have prefix system.i.e. system:kube-scheduler.

For kubelet, it should be the node name. i.e. system:node:node01

**To view certificate**
````
openssl x509 -in apiserver.crt -nout -text
````

## View Logs
To view service logs
````
journalctl -u etcd.service -l
````

# Image
When we write 'image: nginx' in pod template, it means 'library/nginx' . 
````
gcr.io/library/nginx
````
First part can be your registry name
second part is user/account name
third part is image name.

## private registry
- create docker-registry secret
- set imagePullSecrets under pod template
  ````
  imagePullSecrets:
    - name: reg-cred
  ````

## Docker security
To run as non root
````
docker run --user=1000 ubuntu sleep 3600
````
Another way is to put in docker image 'USER 1000'

To add/remove linux capabilities
````
docker run --cap-add=MAC_ADMIN ubuntu sleep 3600
docker run --cap-drop=MAC_ADMIN ubuntu sleep 3600
````
To give all capabilities
````
docker run --previleged ubuntu
````

## Security Context
To set security context, it can be at pod level or container level.
````
securityContent:
  runAsUser: 1000
````
capabilities can be set at container level only
````
securityContext:
  capabilities:
     add: [MAC_ADMIN]
````
