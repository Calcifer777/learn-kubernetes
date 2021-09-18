#####################
Cluster Maintenance
#####################

Draining nodes
*****************

`kubectl cordon node <node-name>`: prevents k8s from scheduling pods on the target node

`kubectl drain node <node-name> --ignore-daemonsets`: `cordon` the target node and move all pods to other nodes

`kubectl uncordon node <node-name>`: allows pod scheduling on the target node

Cluster update process
************************

`Cluster Update process Docs <https://v1-21.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/>`_

At any stage:

- the `kube-apiserver` version is the reference version for the k8s cluster
- the `controller-manager` and the `kube-scheduler` can be at most 1 minor version behind the `kube-apiserver`
- the `kubelet` and the `kube-proxy` can be at most 2 minor versions behind the `kube-apiserver`
- `kubectl` can lag or lead at most 1 minor version w.r.t the `kube-apiserver`

Cluster update strategies
===========================

- All nodes at once
- (Manual) rolling update
- Rolling update with new nodes

Cluster update steps
======================

- Master node(s)
  - Drain the node: `kubectl drain <master-node-name> --ignore-daemonsets` 
  - Update the packages: `apt-get update` 
  - Install the new `kubeadm` version: `apt-get install -y --allow-change-held-packages kubeadm=<version>` 
  - Apply the `kubeadm` update, this will update all the control plane components: `kubeadm update apply <version>` 
  - Install the new `kubectl` and `kube-proxy` executables: `apt-get install -y --allow-change-held-packages kubelet=<version> kubectl=<version>`
  - Restart the kubelet: `sudo systemctl daemon-reload && sudo systemctl restart kubelet`
  - Uncordon the node: `kubectl uncordon <master-node-name>`
- Worker nodes: same as the master node(s), but the drain command must be executed from the master node.

Etcd backup and restore
************************

`Etcd backup and restore <https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster>`_

Steps
======

Backup

- Export the API version environment variable: `export ETCDCTL_API=3`
- Perform the backup to a file; fetch the etcd configuration parameters from the etcd Pod command arguments

  .. code-block:: bash
    
    etcdctl snapshot save <snapshot-file-path> \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/etcd/ca.crt \
        --cert=/etc/etcd/etcd-server.crt \
        --key=/etc/etcd/etcd-server.key

Restore

- Stop the `kube-apiserver`: `service kube-apiserverstop`
- Restore the etcd from the backup:

  .. code-block:: bash
    
    etcdctl snapshot restore <snapshot-file-path> \
        --data-dir=<etcd-backup-data-dir> \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/etcd/ca.crt \
        --cert=/etc/etcd/etcd-server.crt \
        --key=/etc/etcd/etcd-server.key

- Update the etcd static pod definition with the `etcd-backup-data-dir`
- Restart the `daemon` service: `systemctl daemon-reload`
- Restart the `etcd`: `service etcd restart`
- Restart the `kube-apiserver`: `service kube-apiserver start`
