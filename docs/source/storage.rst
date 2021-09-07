
#################
Storage
#################

Docker Storage
*****************

Types of mounts:

- volume mount: mount a volume from the volume directory (`/var/lib/docker/volumes/<volume-name>`)
- bind mount: mount a volume from a directory in the host

.. code-block :: bash

    docker run \
        --volume-driver rexray/ebs \
        --mount type=[bind|mount],source=/path/to/volume,target=/path/in/container/ \
        container-name

Docker uses **storage drivers** to manage the volume layers (AUFS, ZFS, BTRFS, Device Mapper, Overlay, Overlay2).

Volumes are handled by **volume drivers** (Local, AZF, gce-docker, Convoy, RexRay, GlusterFS, etc.)

Kubernetes storage interface
******************************

Defines a set of RPC that must be implemented by the storage driver (e.g. AWS EBS) and can called by the container orchestrator (e.g. K8s) to manage the volumes. 

Volumes
*****************

`Docs <https://kubernetes.io/docs/concepts/storage/volumes/>`_

`Volume` resources decouple the storage from the Container which use them; their lifecycle are coupled to a pod; volumes are therefore transient in nature. Volumes enable safe container restarts and sharing data between containers in a pod. 

Volumes are used to solve the following issues:

- prevent the loss of files when a container crashes 
- sharing files between containers running together in a Pod

K8s supports different types of Volumes, such as:
- hostPath, emptyDir, local
- awsElasticBlockStore, gcePersistentDisk, azureDisk
- cephfs, glusterfs, cinder

.. code-block :: yaml

    apiVersion: v1
    kind: Pod
    metadata: ...
    spec:
      containers:
      - name: boxy1
        ...
        volumeMounts:
        - name: my-volume
          mountPath: /path/in/container/fs
      volumes:
        - name: my-volume
          hostPath:
            path: /path/in/node/fs
            type: Directory

Persistent Volumes
***************************

`PersistentVolume` resources are used to enable persistent storage in a K8s cluster. A `PersistentVolume` decouples the lifecycle of stored data from that of the Pod. Compared to a `Volume`, the lifecycle of a `PersistentVolume` is independent from lifecycle of any Pod in a cluster. 

A `Volume` exists in the context of a pod, that is, you can't create a volume on its own. A `PersistentVolume` on the other hand is a first class object with its own lifecycle, which you can either manage manually or automatically.

Persistent volumes enables safe pod restarts and sharing data between pods.

Examples
==========

.. code-block :: yaml

    apiVersion: v1
    kind: PersistentVolume
    metadata: ...
    spec:
      accessModes: ["ReadOnlyOnce", "ReadOnlyMany", "ReadWriteMany"]
      capacity:
        storage: 1Gi
      awsElasticBlockStore:
        volumeID: volume-id
        fsType: ext4
      persistentVolumeReclaimPolicy: [Retain, Delete, Recycle]

Persisten Volume Claims
***************************

A PersistentVolumeClaim (PVC) is a request for storage by a user. Just like Pods consume node resources (CPU and Memory). Claims can request PersistentVolume resources (storage size and access mode).

.. code-block :: yaml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-claim
    spec:
      accessModes: ["ReadOnlyOnce", "ReadOnlyMany", "ReadWriteMany"]
      resources:
       requests:
         storage: 500Mi

Statuses
===========================

A volume will be in one of the following phases:

- Available: a free resource that is not yet bound to a claim
- Bound: the volume is bound to a claim
- Released: the claim has been deleted, but the resource is not yet reclaimed by the cluster
- Failed: the volume has failed its automatic reclamation

Reclaim Policies
===========================

- Retain: manual reclamation
- Recycle: basic scrub (`rm -rf /thevolume/*`); only valid for NFS and HostPath
- Delete: associated storage asset such as AWS EBS, GCE PD, Azure Disk, or OpenStack Cinder volume is deleted

Classes
===========================

A PV can have a class, which is specified by setting the `storageClassName` attribute to the name of a `StorageClass`. 

A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.

Storage Classes
***************************

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. 

A storageClass is, ultimately, just a label for a re-usable PV profile.

`VolumeBindingMode`: if set to `WaitForFirstConsumer`, will delay the binding and provisioning of a PV until a Pod using the PVC is created.


.. code-block :: yaml

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: my-storage-class
    provisioner: kubernetes.io/no-provisioner
    volumeBindingMode: WaitForFirstConsumer

Dynamic Provisioning
===========================

Dynamic volume provisioning allows storage volumes to be created on-demand. 

The dynamic provisioning feature eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

Dynamic provisioning is enabled by storage classes (but does not work for all provisioners)

.. code-block :: yaml

    apiVersion: storate.k8s.io/v1
    kind: StorageClass
    metadata:
      name: my-storage-class
    provisioner: kubernetes.io/gce-pd  # just an example
    parameters:
      # here go all parameters of a PV from that provisioner
      # in this case, they are relative to gce-pd
      type: pd-standard
      replication-type: none