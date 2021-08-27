
##############
Architecture
##############

The **control plane** is a set of software components that make global decisions about the cluster (e.g. scheduling), as well as detecting and responding to cluster events.

The control plane components are installed only in the master node.

These components are:
- kube-apiserver: exposes the K8s APi
- etcd: key-value database that store the cluster's data
- kube-scheduler: assigns Pods to nodes
- kube-controller-manager: runs controller processes, e.g.

    - Node controller: Responsible for noticing and responding when nodes go down.
    - Job controller 
    - Endpoints controller: joins Services & Pods
    - Service Account & Token controllers: creates default accounts and API access tokens for new namespaces

- cloud-controller-manager: managers cloud-specific cluster components

The **node components** are a set of software components installed in each worker node. They operate the containers in the node, and ensure network connectivity between the pods and the rest of the cluster.

The node components are:
- kubelet: an agent that runs in each node in the cluster; it makes sure that containers are running in a pod. It is also used to register a node to a cluster
- kube-proxy: a network proxy that runs in each node in the cluster
- container runtime: software that is responsible for running containers




