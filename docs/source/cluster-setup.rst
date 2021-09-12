#################
Cluster setup
#################

********************
Cluster design
********************

Purpose
============

Education
------------

- Minikube
- Single node cluster with kubeadm/GCP/AWS 

Development & Testing 
------------------------

- Multi-node cluster with a single master and multiple workers
- Setup using kubeadm tool or quick provision on GCP/AWS/AKS 

Hosting production applications
-----------------------------------

- HA multi node cluster 
- Kubeadm / GKE / Kops (AWS) / AKS

**Storage**

- High performance: SSD backed storage
- Multiple concurrent connections: network based storage
- Shared access across multiple Pods: Persistent shared volumes
- Label nodes with specific disk types
- Use node selectors to assign applications to nodes with specific disk types

**Nodes**

- Virtual or physical machines
- Minimum of 4 node cluster
- Best practice is to not host workloads on master nodes

Architecture
===============

**K8s provider types**

- Turnkey solutions
  - Kops
  - Openshift
  - Cloud Foundry
  - VMware Cloud PKS
  - Vagrant
- Managed solutions
  - GKE
  - AKS
  - Openshift online
  - EKS

**How control plane components work**

- `kube-apiserver`
  - Run in an active-active mode, with a load balancer in front
- `kube-scheduler`, `kube-controller-manager`
  - Run in an active-standby mode
- ETC
  - Can run in a staked (as part of the cluster) or external topology
  - In HA, only the leader processes write operations
  - Uses RAFT as leader election algorithm
  - To ensure HA, use an off number of master nodes (3 or 5)

********************
Kubeadm Installation
********************

kubeadm init \
  --apiserver-advertise-address=10.43.209.3    \
  --apiserver-cert-extra-sans=controlplane  \
  --pod-network-cidr=10.244.0.0/16