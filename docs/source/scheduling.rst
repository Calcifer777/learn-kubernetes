##############
 Scheduling
##############

*******************
Scheduling types
*******************

When deployed, a `Pod` is assigned on a node.

The assignment can run through the control-plane - the scheduler, or manually.

Manual Scheduling
======================

Specifying the `nodeName`
----------------------------

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      - name: c1
        image: nginx
    nodeName: node01

Creating a `Binding` object
----------------------------

.. code-block:: yaml

  apiVersion: v1
  kind: Binding
  metadata:
    name: my-pod
  target:
    apiVersion: v1
    kind: Node
    name: node01

Control-plane scheduling
============================

kube-scheduler
---------------------

A scheduler watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on.

`kube-scheduler` is the default scheduler for Kubernetes and runs as part of the control plane. In `kubeadm` is implemented as a static Pod.

`kube-scheduler` selects a node for the pod in a 2-step operation:

- Filtering: find the set of Nodes where it's feasible to schedule the Pod
- Scoring: rank the remaining nodes to choose the most suitable Pod placement, basing this score on the active scoring rules

By deafult, all Pods are scheduled by the kube-scheduler.

Custom scheduler
---------------------

When a cluster has been deployed with `kubeadm`, a custom scheduler can be created by copying the static pod definition in a node and deploying a new pod by changing the scheduler name (and other options if necessary).

.. code-block:: yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      ...
    schedulerName: custom-scheduler

*******************
Node selectors
*******************

A `nodeSelector` specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels.

The `NodeAuthorizer` is a plugin that prevents the kubelet from changing Pod labels starting with a certain prefix. This helps when using the `nodeSelector` function.

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      ...
    nodeSelector:
      my-key: my-value

*******************
NodeAffinity
*******************

Types:

- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      ...
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorsTerms:
            - matchExpressions:
              - key: some-label-name
                operator: Equals
                value: some-label-value
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1  # used in the final scheduling score
            preference:
              matchExpressions:
              - key: another-node-label-key
                operator: In
                values:
                - another-node-label-value

Inter-pod affinity and anti-affinity
********************************************

Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled based on labels on pods that are already running on the node rather than based on labels on nodes.

These labels (`topologyKey`) usually refer to the topology domain of other pods like the node, cloud region, cloud zone, etc.

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: with-pod-affinity
  spec:
    affinity:
      podAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S1
          topologyKey: topology.kubernetes.io/zone
      podAntiAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
              - key: security
                operator: In
                values:
                - S2
            topologyKey: topology.kubernetes.io/zone

Taints and tolerations
***************************

Taints allow a node to repel a set of pods. You can taint a node with the command:

`kubectl taint node <node-name> <taint-name>=<taint-value>:<taint-effect>`

The effect of a taint - `<taint-effect>` - can be:

- `NoSchedule`: a node without the taint toleration will not be scheduled on that node
- `PreferNoSchedule`: a node without the taint toleration will be assigned a lower score for that node when scheduling when scheduling
- `NoExecute`: a node without the taint toleration will be evicted from a node, even after scheduling

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      ...
    tolerations:
      - key: some-label-key
        operator: "exists"
        effect: "NoSchedule"
        # tolerationSeconds: 6000  # sets the toleration period


Deploy a custom scheduler
*********************************

Copy `kube-scheduler.yaml` from the directory `/etc/kubernetes/manifests/` to any other location and then change the name to my-scheduler.

Add or update the following command arguments in the YAML file:

- `--leader-elect=false`
- `--port=10282`: use of a different port 10282 which is not currently in use in the controlplane.
- `--scheduler-name=my-scheduler`
- `--secure-port=0`: the default scheduler uses secure-port on port `10259` to serve HTTPS with authentication and authorization. This is not needed for our custom scheduler, so we can disable HTTPS by setting the value of secure-port to 0.

Because we have set secure-port to 0, replace HTTPS with HTTP and use the correct ports under liveness and startup probes in the yaml `Pod` configuration file.

The final YAML file would look something like this:

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      component: my-scheduler
      tier: control-plane
    name: my-scheduler
    namespace: kube-system
  spec:
    containers:
    - command:
      - kube-scheduler
      - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
      - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
      - --bind-address=127.0.0.1
      - --kubeconfig=/etc/kubernetes/scheduler.conf
      - --leader-elect=false
      - --port=10282
      - --scheduler-name=my-scheduler
      - --secure-port=0
      image: k8s.gcr.io/kube-scheduler:v1.19.0
      imagePullPolicy: IfNotPresent
      livenessProbe:
        failureThreshold: 8
        httpGet:
          host: 127.0.0.1
          path: /healthz
          port: 10282
          scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      name: kube-scheduler
      resources:
        requests:
          cpu: 100m
      startupProbe:
        failureThreshold: 24
        httpGet:
          host: 127.0.0.1
          path: /healthz
          port: 10282
          scheme: HTTP
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 15
      volumeMounts:
      - mountPath: /etc/kubernetes/scheduler.conf
        name: kubeconfig
        readOnly: true
    hostNetwork: true
    priorityClassName: system-node-critical
    volumes:
    - hostPath:
        path: /etc/kubernetes/scheduler.conf
        type: FileOrCreate
      name: kubeconfig
  status: {}