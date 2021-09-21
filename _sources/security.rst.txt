##############################
Security
##############################

***********************************
Authentication Modes
***********************************

The Kubernetes API server may authorize a request using one of several authorization modes:

- Node: A special-purpose authorization mode that grants permissions to kubelets based on the pods they are scheduled to run
- Attribute-based access control - ABAC: defines an access control paradigm whereby access rights are granted to users through the use of policies which combine attributes together. The policies can use any type of attributes (user attributes, resource attributes, object, environment attributes, etc). ABAC vs RBAC: https://www.dnsstuff.com/rbac-vs-abac-access-control
- Role-based access control (RBAC): is a method of regulating access to resources based on the roles of individual users within an enterprise. In this context, access is the ability of an individual user to perform a specific task, such as view, create, or modify a file. 
- Webhook: a WebHook is an HTTP callback: an HTTP POST that occurs when something happens; a simple event-notification via HTTP POST. A web application implementing WebHooks will POST a message to a URL when certain things happen. 

***********************************
Authentication
***********************************

All user access is managed by the `kube-apiserver`.

Static password file
==============================

A static file with the users' username and password credentials.

The credential file must be passed as an argument to the `kube-apiserver`, or as a Pod argument in the case of `kubeadm`.

.. code-block:: yaml

    apiVersion: v1
    kind: Pod
    metadata:
    name: kube-apiserver
    namespace: kube-system
    spec:
    containers:
      - name: kube-apiserver
        image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
        command:
          - kube-apiserver
          - --authorization-mode=Node,RBAC
            ...
          - --basic-auth-file=/tmp/users/user-details.csv   # <- relevant argument
        volumeMounts:
          - mountPath: /tmp/users
            name: usr-details
            readOnly: true
    volumes:
      - hostPath:
        path: /tmp/users
        type: DirectoryOrCreate
        name: usr-details

Then, for each user, create the necessary roles and role-bindings.

Static token file
==============================

Same as the static password file, but with tokens instead of passwords

Certificates
==============================

Identity services
==============================

***********************************
TLS
***********************************

Types of certificates:

- certificate authority (ca)
- server certificate
- client certificate

Components that use a certificate:

- server
  - kube-api server
  - etcd
  - kubelet
- client
  - admin
  - scheduler
  - kube-controller manager
  - kube-proxy
  - kube-api server (when communicating with the etcd, or the kubelet)
  - kubelet client



******************************
RBAC
******************************

Roles
==============================


.. code-block:: yaml

  kind: Role
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    namespace: default
    name: pod-reader
  rules:
  - apiGroups: [""]                               # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

  ---

  # This role binding allows "jane" to read pods in the "default" namespace.
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: read-pods
    namespace: default
  subjects:
  - kind: User
    name: user1                                   # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role                                    # This must be Role or ClusterRole
    name: pod-reader                              # must match the name of the Role to which to bind
    apiGroup: rbac.authorization.k8s.io


Resources
------------------------------

`Requests Verbs Docs <https://kubernetes.io/docs/reference/access-authn-authz/authorization/#determine-the-request-verb>`_

Service Accounts
==============================

.. code-block:: bash

  kubectl create sa <service-account-name>


Or:

.. code-block:: yaml

  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: <service-account-name>
  secrets:
  - name: <service-account-name>-token-f9gcv


Resources
------------------------------

https://mjarosie.github.io/dev/2021/09/15/iam-roles-for-kubernetes-service-accounts-deep-dive.html


******************************
Kube-config
******************************

A `~/.kube/config` file defines:
- a set of clusters
- a set of users
- a set of contexts; a context defines which user to use for logging in to a cluster

.. code-block:: yaml

  apiVersion: v1
  kind: Config
  clusters:
  - name: my-first-cluster
    cluster:
      certificate-authority: path/to/cert.crt
      server: https://ip-of-the-cluster:port
  - name: ...
  users:
  - name: my-user
  - name: ...
  contexts:
  - name: my-user@my-first-cluster
    context:
      cluster: my-first-cluster
      user: my-user
      namespace: my-first-cluster
  current-context: my-user@my-first-cluster


******************************
Network policies
******************************

NetworkPolicies allow to specify how a `Pod` is allowed to communicate with various network "entities".  These entities are identified through a combination of the following 3 identifiers:

- Other pods that are allowed
- Namespaces that are allowed
- IP blocks

Once there is any `NetworkPolicy` in a namespace selecting a particular `Pod`, that `Pod` will reject any connections that are not allowed by any `NetworkPolicy`.

Network policies are implemented by the network plugin. To use network policies, you must be using a networking solution which supports `NetworkPolicy`.

.. code-block:: yaml

  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: test-network-policy
  spec:
    podSelector:
      matchLabels:
        role: db                  # which Pods the NetworkPolicy applies
    policyTypes:                  # which type of effects the NetworkPolicy will have
    - Ingress
    - Egress
    ingress:
    - from:
      - podSelector:              # allow FROM Pods with given attributes
          matchLabels:
            role: frontend
      - ipBlock:                  # allow FROM specific CIDR blocks
          cidr: 172.17.0.0/16
          except:
          - 172.17.1.0/24
      ports:                      # specify the type of traffic
      - protocol: TCP
        port: 6379
    egress:
    - to:
      - namespaceSelector:        # allow TO a namespace
          matchLabels:
            project: myproject
      ports:
      - protocol: TCP
        port: 5978



******************************
Exercises
******************************

Inspect a certificate
======================================

.. code-block:: bash

  openssl x509 -in <file-path.crt> -text -noout


Generate the certificates for the cluster
===========================================

.. code-block:: bash

  # Create the ca certificate
  open genrsa -out ca.key 2048  # generate the key
  openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr                    # generate the certificate sign request
  openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt                              # sign the certificate

  # Create the client certificate (example for the admin user):

  open genrsa -out admin.key 2048                                                       # generate the key
  openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out ca.csr   # generate the certificate sign request
  openssl x509 -req -in admin.csr -signkey -CA ca.crt -CAkey ca.key -out admin.crt      # sign the certificate with the ca certificate


Create a new user, approve CSR
======================================

Create an user private key and csr
--------------------------------------

.. code-block:: bash

   openssl genrsa -out user-name.key 2048
   openssl req -new -key user-name.key -subj "/CN=user-name" -out user-name.csr

Create a csr K8s object
--------------------------------------

Print the base64 encoded CSR in one line

.. code-block:: bash

  cat user-name.csr | base64 | tr -d '\n'


.. code-block:: yaml

  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: user-name
  spec:
    signerName: kubernetes.ui/kube-apiserver-client
    groups:
      - system:authenticated
    usages:
      - client auth
  requests: <paste-base64-encoded-csr-content>


.. code-block:: bash

  kubectl certificate approve|deny user-name


Inspect the kubeconfig file
======================================

.. code-block:: bash

  kubectl config view --kubeconfig <path-to-config-file>


Use images from private registry
======================================

Create a secret with the credentials to the private repository

.. code-block:: bash

  kubectl create secret docker-registry <docker-registry-secret-name>  \   # use this in the next step
    --docker-username=user  \
    --docker-password=password  \
    --docker-email=email  \
    --docker-server=string


Edit the corresponding entry in the `Pod` definition

.. code-block:: yaml

  ...
  spec:
    imagePullSecrets:
    - name: docker-registry-secret-name
    containers:
    - ...


Set containers security context properties
===============================================

You can set the security context at the Pod level or at the container level. Specifications at the container level overwrite those at the `Pod` level.

.. code-block:: yaml

  ...
  securityContext:
    runAsUser: my-user
  spec:
    containers:
    - securityContext
        runAsUser: my-other-user  # this has precedence over the Pod level specification
        capabilities:
          add:
            - SYS_TIME


Useful commands
=======================


.. code-block:: bash
  
  kubectl api-resources  # display all Kubernetes resource types
