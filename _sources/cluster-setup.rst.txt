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

.. code-block:: bash
  
  export VERSION=1.21.0-00

  # KUBEADM

  # Update the package manager
  sudo apt-get update
  sudo apt-get install -y apt-transport-https ca-certificates curl
  # Download the Google Cloud public signing key
  sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
  # Add the K9s repository
  echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  # Install kubeadm, kubelet, and kubectl
  sudo apt-get update
  sudo apt-get install -y kubeadm=${VERSION} kubelet=${VERSION} kubectl=${VERSION}
  # Check versions
  kubeadm version
  kubelet --version
  kubectl version


  # MASTER NODE
  export ADVERTISE_ADDRESS=$(ip addr | grep eth0 | grep inet | sed 's/inet\s\([0-9\.]*\).*/\1/')
  # Install the cluster
  kubeadm init \
    --apiserver-advertise-address=10.31.173.12  \
    --apiserver-cert-extra-sans=controlplane  \
    --pod-network-cidr=10.244.0.0/16
  # Create default kubeconfig
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  export KUBECONFIG=/etc/kubernetes/admin.conf
  # Install network plugin (Flannel)
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  # Create join token
  kubeadm token create --print-join-command


  # WORKER NODES
  # print join-command from master node
