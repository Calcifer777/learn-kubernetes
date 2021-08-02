
##############
 Workloads
##############

ReplicaSets
**************

A `ReplicaSet` ensures that a specified number of pod replicas are running at any given time. 

It is a deprecated version of a `Deployment`; use Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.


Example
**************

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
