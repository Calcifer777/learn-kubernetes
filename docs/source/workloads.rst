

==============
 Workloads
==============

Pods
--------------

`Docs <https://kubernetes.io/docs/concepts/workloads/pods/>`_

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared contex

Example
``````````````

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-example
  spec:
    containers:
    - name: ubuntu
      image: ubuntu:trusty
      command: ["echo"]
      args: ["Hello World"]

ReplicaSets
-------------

A `ReplicaSet` ensures that a specified number of pod replicas are running at any given time. 

It is a deprecated version of a `Deployment`; use Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.


Example
``````````````

.. code-block:: yaml

  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: replicaset-example
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14
