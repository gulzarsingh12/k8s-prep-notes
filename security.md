# Security primitivies

There are multiple things we may need to do to secure a cluster. Some of them can be as below
* To secure as node/vm - We can restrict the access to machine/nodes by setting up the ssh auth key and disabling password based authentication to login.
* To secure the cluster
  * Authorization - We can restict the access to the used using RBAC, ABAC, Node Authorizer, Webhook mode
  * Authentication - we can use certificates, LDAP etc for users. we use service accounts for applications.
* Data in transit
  We can use TLS certificates for all communications to encrypt the data in transit
* Data at rest
  we already see in etcd section how to configure the secret key in encyption config to ecrypt the data by kube apiserver while saving to etcd.
* Incoming/Outgoing access to pods
  We use network policy to restrict access between pods
