
##############
IAM
##############

API groups
************

kubectl proxy  # https proxy server to acces the kubeapi-server without furher authentication
curl http://localhost:8001 -k

Authorization types 
********************

- Node: any request coming from a user with the name system-node and part of the system-nodes group is authorized by the node-authorizer and granted priviledges
- ABAC: configure a list of grants for each user
- RBAC: define a role with a set of grants and associate a user to that role
- Hooks: leverage a third party IAM tool to verify priviledges

When multiple types of requests authorization are configured, each type of authorization is checked for each request; if any allows the request, the priviledge is granted.

Roles and Role bindings
***************************

.. code-block :: yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata: 
      name: developer
    rules:
    - apiGroups: ["*"]
      resources: ["pods"]
      verbs: ["list", "get", "create", "update"]


.. code-block :: yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata: 
      name: developer-developer-binding
    subjects:
    - kind: user
      name: dev-user
      apiGroup: rbac.authorization.k8s.io/v1
    roleRef:
        kind: Role
        name: developer
        apiGroup: rbac.authorization.k8s.io/v1


`kubectl auth can-it <action> <resource> --as <user-name>`

Cluster roles 
**************

Resources are categorized as:
- namespaced
- cluster scoped

Service accounts
*****************

Types of accounts:
- user accounts
- service accounts

A service account can be linked to a role to give it the necessary permissions.

A service account creates a secret with a token that can be used to authenticate
the requests to the kubeapi-server. The secret can then be used as a volume in the container that needs it.

Image security
*****************

To pull images from a private repository:

- Create a secret of type `docker-registry`:

  .. code-block :: bash

    kubectl create secret docker-registry regcred \
      --docker-server=private-registry.io \
      --docker-username=registry-user \
      --docker-password=registry-password \
      --docker-email=registry-user@org.com

- Specify the secret  in the Pod specification:

  .. code-block :: bash

    apiVersion: v1
    kind: Pod
    metadata:
      ...
    spec:
      containers:
      - ...
        imagePullSecrets:
        - name: regcred

Security contexts
*****************

You can choose to configure the security settings at a Pod level or at a container level.

The settings in the container override the settings in the Pod.

Network Policies
*****************