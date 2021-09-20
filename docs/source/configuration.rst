#################
Configuration
#################

ConfigMaps
*****************

`Docs <https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/config-map-v1/>`_

A `ConfigMap` is an API object used to store non-confidential data in key-value pairs.

The data stored in a `ConfigMap` cannot exceed 1 MiB.

There are four different ways that you can use a `ConfigMap` to configure a container inside a `Pod`:

- Inside a container command and args
- Environment variables for a container
- Add a file in read-only volume, for the application to read
- Write code to run inside the Pod that uses the Kubernetes API to read a `ConfigMap`: allows to access a `ConfigMap` in different namespaces, monitor `ConfigMap` changes

When a `ConfigMap` currently consumed in a volume is updated, projected keys are eventually updated as well (1 minute by default).

Useful commands
=================

.. code-block::

  # From file
  kubectl create configmap my-cm-1  --from-file=./properties.cfg
  # From file with custom key
  kubectl create configmap my-cm-2  --from-file=custom-key=./properties.cfg
  # From env file
  kubectl create configmap my-cm-3  --from-env-file=./cm-env-file

Example
=================

.. code-block:: yaml

  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config-map
  data:
    key1: value1  # keys and values are in plain text
    key2: value2
  binaryData:       # can co-exist with the data field
    key1: dmF1ZTEK  # keys and values are bas64 encoded
    key2: dmF1ZTIK
  immutable: true

Secrets
*****************

`Docs <https://kubernetes.io/docs/concepts/configuration/secret/>`_

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.

Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). In order to safely use Secrets, take at least the following steps:

- Enable Encryption at Rest for Secrets.
- Enable or configure RBAC rules that restrict reading data in Secrets

A Secret can be used with a Pod in three ways:

- As files in a volume mounted on one or more of its containers.
- As container environment variable.
- By the kubelet when pulling images for the Pod.

When a secret currently consumed in a volume is updated, projected keys are eventually updated as well.

Types of Secret
=================

+-------------------------------------+---------------------------------------+
| Type                                | Usage                                 |
+=====================================+=======================================+
| Opaque                              | arbitrary user-defined data           |
+-------------------------------------+---------------------------------------+
| kubernetes.io/service-account-token | service account token                 |
+-------------------------------------+---------------------------------------+
| kubernetes.io/dockercfg             | serialized ~/.dockercfg file          |
+-------------------------------------+---------------------------------------+
| kubernetes.io/dockerconfigjson      | serialized ~/.docker/config.json file |
+-------------------------------------+---------------------------------------+
| kubernetes.io/basic-auth            | credentials for basic authentication  |
+-------------------------------------+---------------------------------------+
| kubernetes.io/ssh-auth              | credentials for SSH authentication    |
+-------------------------------------+---------------------------------------+
| kubernetes.io/tls                   | data for a TLS client or server       |
+-------------------------------------+---------------------------------------+
| bootstrap.kubernetes.io/token       | bootstrap token data                  |
+-------------------------------------+---------------------------------------+

Example
=================


.. code-block:: yaml

  apiVersion: v1
  kind: Secret
  metadata:
    name: sample-secret
  stringData:
    key: value
  data:
    asdasd=: asglaf=  # base64 encoded

Requests and Limits
***********************

When you specify the resource request for Containers in a Pod, the scheduler uses this information to decide which node to place the Pod on. 

When you specify a resource limit for a Container, the kubelet enforces those limits so that the running container is not allowed to use more of that resource than the limit you set.

If a Container specifies its own memory/CPU limit, but does not specify a memory/CPU request, Kubernetes automatically assigns a memory/CPU request that matches the limit.

Units
==============

Memory is measured in: XX KiB, XX MiB

CPU: 0.X -> X00m (of a vCPU/core/hyperthread)

Remedies
==============

If a Container exceeds its memory limit, it might be terminated. If it is restartable, the kubelet will restart it.

If a Container exceeds its memory request, it is likely that its Pod will be evicted whenever the node runs out of memory.

A Container might or might not be allowed to exceed its CPU limit for extended periods of time. However, it will not be killed for excessive CPU usage.

CPU limits are enforced via Linux control groups.

QoS classes
=============

When Kubernetes creates a `Pod` it assigns one of these QoS classes to the Pod:

- Guaranteed: for all containers in the `Pod`, both requests and limits are specified and set to the same value
- Burstable: at least one container in the `Pod` has memory or CPU request
- BestEffort: for all containers in the `Pod` neither requests or limits are specified

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend
  spec:
    containers:
    - resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"