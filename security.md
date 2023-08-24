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
