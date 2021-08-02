##############
 Pods
##############


`Docs <https://kubernetes.io/docs/concepts/workloads/pods/>`_

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. 

Pods natively provide two kinds of shared resources for their constituent containers:
- storage: a Pod can specify a set of shared storage volumes
- networking: inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using `localhost`

Any container in a Pod can enable **privileged mode**, using the privileged flag on the security context of the container spec. This is useful for containers that want to use operating system administrative capabilities such as manipulating the network stack or accessing hardware devices.

Static Pods
**************

Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Whereas most Pods are managed by the control plane (for example, a Deployment), for static Pods, the kubelet directly supervises each static Pod (and restarts it if it fails).

Kubelet also tries to create a mirror pod on the kubernetes api server for each static pod so that the static pods are visible i.e., when you do kubectl get pod for example, the mirror object of static pod is also listed.

The main use for static Pods is to run a self-hosted control plane: in other words, using the kubelet to supervise the individual control plane components. For example, when kubeadm is bringing up kubernetes control plane, it generates pod manifests for api-server and controller-manager in a directory which kubelet is monitoring. Then kubelet brings up these control plane components as static pods.

Pod Lifecycle
**************

Pod Phases
==============

- `Pending`: The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network.
- `Running`: The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.
- `Succeeded`: All containers in the Pod have terminated in success, and will not be restarted.
- `Failed`: All containers in the Pod have terminated, and at least one container has terminated in failure.
- `Unknown`: For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.
- `Terminated` (not a real Pod phase): When a Pod is being deleted, it is shown as Terminating by some kubectl commands

Container states
==================


- `Waiting`: the container is still running the operations it requires in order to complete start up
- `Running`: the container is executing without issues. If there was a `postStart` hook configured, it has already executed and finished.
- `Terminated`: the container began execution and then either ran to completion or failed for some reason. If a container has a `preStop` hook configured, that runs before the container enters the Terminated state.

**Container restart policy**

- Always
- OnFailure
- Never

Pod status
==================

The PodStatus, which has an array of PodConditions through which the Pod has or has not passed:
- `PodScheduled`: the Pod has been scheduled to a node.
- `ContainersReady`: all containers in the Pod are ready.
- `Initialized`: all init containers have started successfully.
- `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services.

Readiness Probe
==================

Your application can inject extra feedback or signals into PodStatus: Pod readiness. To use this, set readinessGates in the Pod's spec to specify a list of additional conditions that the kubelet evaluates for Pod readiness.


Container Probes
------------------

A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet can invoke different actions:

- `ExecAction` (performed with the help of the container runtime)
- `TCPSocketAction` (checked directly by the kubelet)
- `HTTPGetAction` (checked directly by the kubelet)

The kubelet can optionally perform and react to three kinds of probes on running containers:

- `startupProbe`: Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a Container does not provide a startup probe, the default state is Success.
- `readinessProbe`: Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is Failure. If a Container does not provide a readiness probe, the default state is Success.
- `livenessProbe`: Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container.

Special types of Containers
******************************

Init containers
=================

Init containers are exactly like regular containers, except:

- Init containers always run to completion.
- Each init container must complete successfully before the next one starts.

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds.

**Use cases**

- Wait for a Service to be created (e.g. a db, another microservice, etc)
- Register this Pod with a remote server from the downward API
- Wait for some time before starting the app container with a command like
- Clone a Git repository into a Volume
- Place values into a configuration file and run a template tool to dynamically generate a configuration file for the main app container. 

Ephemeral containers (alpha)
==============================


Ephemeral containers are a special type of container that runs temporarily in an existing Pod to accomplish user-initiated actions such as troubleshooting. Ephemeral containers are useful for interactive troubleshooting when kubectl exec is insufficient because a container has crashed or a container image doesn't include debugging utilities (e.g. distroless images).

You use ephemeral containers to inspect services rather than to build applications.

Topology spread constraints
******************************

Spread constraints for pods
==============================

You can use topology spread constraints to control how Pods are spread across your cluster among failure-domains such as regions, zones, nodes, and other user-defined topology domains.

Topology spread constraints rely on node labels to identify the topology domain(s) that each Node is in. For example, a Node might have labels: `node=node1`, `zone=us-east-1a`, `region=us-east-1`

You can define one or multiple `topologySpreadConstraint` to instruct the kube-scheduler how to place each incoming Pod in relation to the existing Pods across your cluster. The fields are:

- `labelSelector` is used to find matching Pods. Pods that match this label selector are counted to determine the number of Pods in their corresponding topology domain
- `topologyKey` is the key of node labels
- `maxSkew`: the degree to which Pods may be unevenly distributed. It must be greater than zero. Its semantics differs according to the value of `whenUnsatisfiable`
- `whenUnsatisfiable`: indicates how to deal with a Pod if it doesn't satisfy the spread constraint
    - `DoNotSchedule`
    - `ScheduleAnyway`: schedules the pod prioritizing nodes that minimize the skew.

When a Pod defines more than one `topologySpreadConstraint`, those constraints are `ANDed`.

Cluster-level default constraints
====================================

It is possible to set default topology spread constraints for a cluster. Default topology spread constraints are applied to a Pod if, and only if:

- It doesn't define any constraints in its .spec.topologySpreadConstraints.
- It belongs to a service, replication controller, replica set or stateful set.


Pods disruptions
***********************

How to deal with involuntary disruptions:

- Ensure your pod requests the resources it needs.
- Replicate your application if you need higher availability. (Learn about running replicated stateless and stateful applications.)
- For even higher availability when running replicated applications, spread applications across racks (using anti-affinity) or across zones (if using a multi-zone cluster.)

Pod disruptions budgets (PDB)
================================

A PDB limits the number of Pods of a replicated application that are down simultaneously from voluntary disruptions. PDBs cannot prevent involuntary disruptions from occurring, but they do count against the budget.

Pods which are deleted or unavailable due to a rolling upgrade to an application do count against the disruption budget, but workload resources (such as Deployment and StatefulSet) are not limited by PDBs when doing rolling upgrades. Instead, the handling of failures during application updates is configured in the spec for the specific workload resource. 

Example
****************

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
    initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]


