#####################
Cluster Maintenance
#####################

*****************************
Useful commands
*****************************

Draining nodes
=============================

`kubectl cordon node <node-name>`: prevents k8s from scheduling pods on the target node

`kubectl drain node <node-name> --ignore-daemonsets`: `cordon` the target node and move all pods to other nodes

`kubectl uncordon node <node-name>`: allows pod scheduling on the target node

*****************************
Cluster update process
*****************************

Version skew constrains
=============================

`Cluster Update process Docs <https://v1-21.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/>`_

Kubernetes versions are expressed as `x.y.z`, where `x` is the major version, `y` is
the minor version, and `z` is the patch version, following Semantic Versioning
terminology.

Each Kubernetes version supports the previous 2 minor versions.

At any stage:

- the `kube-apiserver` version is the reference version for the Kubernetes cluster
- the `controller-manager` and the `kube-scheduler` can be at most 1 minor version behind the `kube-apiserver`
- the `kubelet` and the `kube-proxy` can be at most 2 minor versions behind the `kube-apiserver`
- `kubectl` can lag or lead at most 1 minor version with respect to the `kube-apiserver`

The recommended approach is to update 1 minor version at a time.

Cluster update strategies
=============================

- All nodes at once
- (Manual) rolling update
- Rolling update with new nodes

Cluster update steps
=============================

Master Node(s)
-----------------------------

.. code-block:: yaml

  # Mark the node as unschedulable and drain it
  kubectl cordon <master-node-name>
  kubectl drain <master-node-name> --ignore-daemonsets

  export KUBEADM_VERSION=1.20.0-00
  export K8S_VERSION= $(echo $KUBEADM_VERSION | sed 's/\([0-9\.]\)-\(.*\)/\1/')  # KUBEADM_VERSION without the "-00" suffix

  # Check which new K8s versions are available
  kubeadm upgrade plan | grep -i "latest stable version"

  # Update the packages: 
  apt-get update

  # Install the new `kubeadm` version
  apt-get install -y --allow-change-held-packages kubeadm=<version>

  # Apply the `kubeadm` update, this will update all the control plane components
  kubeadm update apply <version> 

  # Install the new `kubectl` and `kube-proxy` executables
  apt-get install -y --allow-change-held-packages kubelet=<version> kubectl=<version>

  # Restart the kubelet
  sudo systemctl daemon-reload && sudo systemctl restart kubelet

  # Uncordon the node
  kubectl uncordon <master-node-name>


Worker nodes
-----------------------------

Similar to the master nodes, but the `cordon` and `drain` commands must be executed from the master node.

From one of the master nodes

.. code-block:: yaml

  kubectl cordon <worker-node-name>
  kubectl drain <worker-node-name>  --ignore-daemonsets --grace-period 1


From the worker node

.. code-block:: yaml

  export KUBEADM_VERSION=1.20.0-00

  apt get update

  # Upgrade kubeadm
  apt-get install -y --allow-change-held-packages kubeadm=${KUBEADM_VERSION}

  # Align node to the master version
  kubeadm upgrade node

  # Upgrade the kubelet and kubectl
  apt-get install -y --allow-change-held-packages \
    kubelet=${KUBEADM_VERSION}  \
    kubectl=${KUBEADM_VERSION}

  # Restart the kubelet
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet


*****************************
ETCD backup and restore
*****************************

`Etcd backup and restore <https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster>`_

Backup
=============================

  .. code-block:: bash

    #!/bin/bash
    
    export ETCDCTL_API="3"
    export ENDPOINT=https://127.0.0.1:2379
    export SNAPSHOT_FILE_PATH=/opt/snapshot-pre-boot.db
    export CA_CERT_FILE_PATH=/etc/kubernetes/pki/etcd/ca.crt
    export CERT_FILE_PATH=/etc/kubernetes/pki/etcd/server.crt
    export KEY_FILE_PATH=/etc/kubernetes/pki/etcd/server.key

    # Snapshot the ETCD database
    etcdctl  \
      --endpoints=${ENDPOINT}  \
      --cacert=${CA_CERT_FILE_PATH}  \
      --cert=${CERT_FILE_PATH}  \
      --key=${KEY_FILE_PATH}  \
      snapshot save ${SNAPSHOT_FILE_PATH}

    # Verify snapshot, print its contents
    etcdctl --write-out=table snapshot status ${SNAPSHOT_FILE_PATH}
    

Restore
=============================

(OPTIONAL) Stop the kube aposerver
---------------------------------------

.. code-block:: bash

  service kube-apiserver stop


Restore the ETCD from the backup
---------------------------------------

.. code-block:: bash
  
  etcdctl snapshot restore <snapshot-file-path> \
    --data-dir=<etcd-backup-data-dir> \            # use this in the next step
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/etcd/ca.crt \
    --cert=/etc/etcd/etcd-server.crt \
    --key=/etc/etcd/etcd-server.key


Update ETCD manifest
---------------------------------------

Update the etcd static pod definition with the `etcd-backup-data-dir` for the
corresponding `volume`

.. code-block:: bash

 ...
  volumes:
  - hostPath:
      path: <etcd-backup-data-dir>
      type: DirectoryOrCreate
    name: etcd-data


(OPTIONAL) Restart ETCD service
---------------------------------------

.. code-block:: bash

  ...
  # Restart the daemon service
  systemctl daemon-reload
  # Restart the etcd service 
  service etcd restart
  # Restart the kube-apiserver
  service kube-apiserver start
