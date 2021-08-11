
#################
Storage
#################

Volumes
*****************

`Docs <https://kubernetes.io/docs/concepts/storage/volumes/>`_

`Volume` resources decouple the storage from the Container which use them; their lifecycle are coupled to a pod. Volumes enable safe container restarts and sharing data between containers in a pod. 

Volumes are used to solve the following issues:

- prevent the loss of files when a container crashes 
- sharing files between containers running together in a Pod

K8s supports different types of Volumes, such as:
- hostPath, emptyDir, local
- awsElasticBlockStore, gcePersistentDisk, azureDisk
- cephfs, glusterfs, cinder


Persistent Volumes
***************************

`PersistentVolume` resources are used to enable persistent storage in a K8s cluster. A `PersistentVolume` decouples the storage from the Pod. Compared to a `Volume`, the lifecycle of a `PersistentVolume` is independent from lifecycle of any Pod in a cluster. 

A `Volume` exists in the context of a pod, that is, you can't create a volume on its own. A `PersistentVolume` on the other hand is a first class object with its own lifecycle, which you can either manage manually or automatically.

Persistent volumes enables safe pod restarts and sharing data between pods.

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

Persisten Volume Claims
***************************

A PersistentVolumeClaim (PVC) is a request for storage by a user. Just like Pods consume node resources (CPU and Memory). Claims can request PersistentVolume resources (storage size and access mode).

Storage Classes
***************************

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. 

A storageClass is, ultimately, just a label for a re-usable PV profile.


Dynamic Provisioning
===========================

Dynamic volume provisioning allows storage volumes to be created on-demand. 

The dynamic provisioning feature eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.
