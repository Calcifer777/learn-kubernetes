
##############
 Workloads
##############

**************
ReplicaSets
**************

A `ReplicaSet` ensures that a specified number of pod replicas are running at any given time. 

It is a deprecated version of a `Deployment`; use Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

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


*****************
Deployment
*****************

Deployment Strategies
==========================


- Recreate: terminate the old version and release the new one

  .. code-block:: yaml

    ... 
    spec:
      strategy:
        type: Recreate
    ...

- Ramped: release a new version on a rolling update fashion, one after the other

  .. code-block:: yaml

    ... 
    spec:
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: <max-unavail>  # how many pods can be unavailable during the rolling update
          maxSurge: <max-surge>  # how many pods we can add at a time
    ...

- Blue/Green: the new application version  is deployed alongside the old version with exactly the same amount of instances. After testing that the new version meets all the requirements the traffic is switched from the old version to the new version at the load balancer level.

  - How to: Create a new deployment targeting a different application version/tag, and with a different selector
  - Pro's

    - instant rollout/rollback
    - avoid versioning issue, change the entire cluster state in one go

  - Con's

    - requires double the resources
    - proper test of the entire platform should be done before releasing to production
    - handling stateful applications can be hard

- Canary: release a new version to a subset of users, then proceed to a full rollout

  - How: create 2 deployments and split traffic at the load balancer level (HAProxy, Linkerd, etc.)
  - Pro's

    - version released for a subset of users
    - convenient for error rate and performance monitoring
    - fast rollback

- A/B testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.). A/B testing is really a technique for making business decisions based on statistics but we will briefly describe the process. This doesn’t come out of the box with Kubernetes, it implies extra work to setup a more advanced infrastructure (Istio, Linkerd, Traefik, custom nginx/haproxy, etc).
  
  - Pro

    - requires intelligent load balancer
    - several versions run in parallel
    - full control over the traffic distribution

  - Cons

    - hard to troubleshoot errors for a given session, distributed tracing becomes mandatory
    - not straightforward, you need to setup additional tool


Horizontal Pod Autoscaling
===============================

Editing Deployments
=======================

When applying any changes to a `Deployment`, you can save the change in the annotations using the `--record` cli option.

.. code-block:: bash

  kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
  kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>

Rollouts
=================

Rollouts commands are useful to get the state of a deployment:

- `kubectl rollout status <deployment-name>`
- `kubectl rollout history <deployment-name> [ --revision <revision-history-index> ]`

  .. code-block::

    deployments "nginx-deployment" revision 2
    Labels:       app=nginx
            pod-template-hash=1159050644
    Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    Containers:
     nginx:
      Image:      nginx:1.16.1
      Port:       80/TCP
       QoS Tier:
          cpu:      BestEffort
          memory:   BestEffort
      Environment Variables:      <none>
    No volumes.

- `kubectl rollout undo <deployment-name> [ --to-revision <revision-history-index> ]`: restore the deployment to the `revision-history-index` version
