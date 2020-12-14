# OpenShift Container Storage (WIP)

[OpenShift Container Storage v4](https://www.openshift.com/products/container-storage/) - or OCS for short - is persistent software-defined storage integrated with and optimized for Red Hat OpenShift Container Platform.

## Resource Requirements

OCS has certain resource requirements which are to be fulfilled, see [Node Requirements](https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.5/html-single/planning_your_deployment/) in the product documentation.

> For this workshop, available lab environments hardly can fulfil these resource requirements, we therefore overprovision the hypervisor to obtain a running system demonstrating OCS setup and configuration, allow for lightweight usage. This setup is not suitable beyond this workshop!

## Setup/Extend Workshop environment

We extend the existing OpenShift cluster setup by 3 additional (worker) nodes which will be configured as storage nodes for OCS.

### Variant 1 - Scaleout cluster

Perform a scaleout of the cluster with additional worker nodes as described in the respective module. 

Use the following resource settings for each of the three nodes

* 16 vCPU (overprovision the hypervisor)
* 32 GB RAM per node

> Note: one can modify resource settings of provisioned VMs on the hypervisor by
> 
> * gather VM name using `virsh list` (VM is called domain in 'virsh')
> * shutting down the VM `virsh shutdown domain`
> * editing the VM `virsh edit domain`and set CPU and memory
> * (re)starting the VM `virsh start domain`

### Variant 2 - Size existing worker or infra nodes

For the workshop, we can use existing worker or infrastructure nodes as well for OCS. These nodes must be adjusted for node resources to fit the OCS requirements.

Use the following resource settings for each of the three nodes

* 16 vCPU (overprovision the hypervisor)
* 64 GB RAM per node

> Note: one can modify resource settings of provisioned VMs on the hypervisor by
> 
> * gather VM name using `virsh list` (VM is called domain in 'virsh')
> * shutting down the VM `virsh shutdown domain`
> * editing the VM `virsh edit domain`and set CPU and memory
> * (re)starting the VM `virsh start domain`

### Nodes being used

For the remainder of this module, we assume the following naming for the OCS nodes:

* virsh VM (domain) names: ocp4-compute-3, ocp4-compute-4, ocp4-compute-5
* OpenShift node names: compute-3, compute-4, compute-5

### Create disks for OCS

On the Hypevisor, create disks for OCS. Each OCS node requires a single disk of equal size.

We create two disks per node, as we want to demonstrate the extension of OCS storage in a later section of this module.

#### Create the disk images

Adjust the size of the disks to your needs.

```shell
cd /var/lib/libvirt/images/
qemu-img create -f raw rhcos-compute3-vm-ocs1-disk 100G
qemu-img create -f raw rhcos-compute4-vm-ocs1-disk 100G
qemu-img create -f raw rhcos-compute5-vm-ocs1-disk 100G
qemu-img create -f raw rhcos-compute3-vm-ocs2-disk 50G
qemu-img create -f raw rhcos-compute4-vm-ocs2-disk 50G
qemu-img create -f raw rhcos-compute5-vm-ocs2-disk 50G
```

#### Attach disks to the nodes

Stop the nodes

```shell
virsh shutdown ocp4-compute-3
virsh shutdown ocp4-compute-4
virsh shutdown ocp4-compute-5
```

Attach the disks

```shell
virsh attach-disk ocp4-compute-3 --source /var/lib/libvirt/images/rhcos-compute3-vm-ocs1-disk --target vdb --persistent
virsh attach-disk ocp4-compute-4 --source /var/lib/libvirt/images/rhcos-compute4-vm-ocs1-disk --target vdb --persistent
virsh attach-disk ocp4-compute-5 --source /var/lib/libvirt/images/rhcos-compute5-vm-ocs1-disk --target vdb --persistent
virsh attach-disk ocp4-compute-3 --source /var/lib/libvirt/images/rhcos-compute3-vm-ocs2-disk --target vdc --persistent
virsh attach-disk ocp4-compute-4 --source /var/lib/libvirt/images/rhcos-compute4-vm-ocs2-disk --target vdc --persistent
virsh attach-disk ocp4-compute-5 --source /var/lib/libvirt/images/rhcos-compute5-vm-ocs2-disk --target vdc --persistent
```

Start the nodes

```shell
virsh start ocp4-compute-0
virsh start ocp4-compute-1
virsh start ocp4-compute-2
```

and wait for the nodes be 'ready' again.

## Mark Nodes as Storage Nodes

### Label Nodes

OCS requires specific node labels. Label the nodes accoring to OCS prerequisites:

```
[root@bastion ~]# oc label nodes compute-0 cluster.ocs.openshift.io/openshift-storage=''
[root@bastion ~]# oc label nodes compute-1 cluster.ocs.openshift.io/openshift-storage=''
[root@bastion ~]# oc label nodes compute-2 cluster.ocs.openshift.io/openshift-storage=''
```

### Create MachineConfigPool for Storage Nodes

In addition to the labels, we create a machine config pool to create storage nodes similar to creating infrastructure nodes.

#### Create a new MachineConfigPool storage

```yaml
---
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: storage
spec:
  machineConfigSelector:
    matchExpressions:
    - key: machineconfiguration.openshift.io/role
      operator: In
      values:
      - worker
      - storage
  maxUnavailable: 1
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/storage: ""
  paused: false
...
```

```
[root@bastion ~]# oc create -f storage-machineconfigpool.yaml
```

#### Relabel Node-Roles of Worker machines as Storage Nodes

For each of the nodes, add the 'storage'  role and remove the 'worker' role

```shell
for node in $(oc get nodes --no-headers -l cluster.ocs.openshift.io/openshift-storage -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}'); \
do \
  echo $node; \
  oc label node $node node-role.kubernetes.io/storage=  ; \
  oc label node $node node-role.kubernetes.io/worker- ; \
done
```

Wait for the Changes to get applied by the MachineConfigOperator.

## Install Local Volume Operator

We use the Local Volume Operator to add the disks added on Hypervisor level to the VMs as local storage.

### Create the namespace

The local volume operator needs its own namespace.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: local-storage
  labels:
    openshift.io/cluster-monitoring: 'true'
...
```

```shell
oc create -f namespace-local-storage.yaml
```

### Create the Operatorgroup

```yaml
---
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: local-operator-group
  namespace: local-storage
spec:
  targetNamespaces:
    - local-storage
...
```

```shell
oc create -f operatorgroup-local-storage.yaml
```

### Create the Operator Subscription

Check which software version of the operator is available

```shell
oc get packagemanifest local-storage-operator -n openshift-marketplace \
  --template='{{range .status.channels}}{{.name}}{{"\n"}}{{end}}'
```

Replace `<SOFTWARE-VERSION>` in 'subscription-local-storage.yaml' and create it. E.g. replace `<SOFTWARE-VERSION>` by `4.5`.

```yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: local-storage-operator
  namespace: local-storage
spec:
  channel: '<SOFTWARE-VERSION>'
  installPlanApproval: Automatic
  name: local-storage-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
...
```

```shell
oc create -f subscription-local-storage.yaml
```

#### Verify the installation

Check Pods - the pod for the Operator must be rolled out.

```shell
oc get pods -n local-storage
```

Check the ClusterServiceVersion (csvs) manifest

```shell
oc get csvs -n local-storage
```

## Create LocalVolumes for OCS

To determine the configuration of the local volume, we must find out the exact device / device path of the disks on each node.

```shell
for node in $(oc get nodes --no-headers -l cluster.ocs.openshift.io/openshift-storage -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}'); do echo $node; oc debug node/$node -- chroot /host ls -l /dev/disk/by-id; oc debug node/$node -- chroot /host lsblk; done
```

The above command will list for each storage node the block devices (using `lsblk` command) and shows the disk id device path.

> Note: For the workshop 'libvirt' environment we need to use static `/dev/xyz` devices. For other environments you likely will see the disks appearing in `/dev/disk/by-id`. As the latter is fixed accross machine reboots, use these path information, if available.

 h
Apply the file 

```shell
oc create -f localvolume-ocs.yaml
```

and wait for the pods managing the disks to appear. 2 Pods per device will be available.

```shell
oc get pods -n local-storage
```

Run as well the following commands:

```shell
oc get pv
oc get sc
```

There should be a StorageClass 'ocs-localblock' and 3 PVs 'local-pv-....' for that StorageClass being created.

## Deploy OpenShift Container Storage Operator

### Create Namespace

OCS must reside in a namespace having a given name. Do not change this name.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-storage
  labels:
    openshift.io/cluster-monitoring: 'true'
...
```

```shell
oc create -f namespace-ocs-storage.yaml
```

### Create OperatorGroup

```yaml
---
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  generateName: openshift-storage-
  namespace: openshift-storage
spec:
  targetNamespaces:
    - openshift-storage
...
```

```shell
oc create -f operatorgroup-openshift-storage.yaml
```

### Create Operator Subscription

Check which software version of the operator is available

```shell
oc get packagemanifest ocs-operator -n openshift-marketplace \
  --template='{{range .status.channels}}{{.name}}{{"\n"}}{{end}}'
```

Replace `<SOFTWARE-VERSION>` in 'subscription-local-storage.yaml' and create it. E.g. replace `<SOFTWARE-VERSION>` by `stable-4.5`.

> Note: OCS v4.4 can be installed on OCP v4.5.

```yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ocs-operator
  namespace: openshift-storage
spec:
  channel: '<SOFTWARE-VERSION>'
  installPlanApproval: Automatic
  name: ocs-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
...
```

```shell
oc create -f subscription-openshift-storage.yaml
```

### Verify Installation

Check Pods - the pod for the Operator must be rolled out.

```shell
oc get pods -n openshift-storage
```

Check the ClusterServiceVersion (csvs) manifest

```shell
oc get csvs -n openshift-storage
```

Wait for the Operator to be fully rolled-out.

## Create OCS Storage Cluster

### Create the Storage Cluster Configuration

Use the following file as the definition of the Storage Cluster to be rolled-out.

> Note: The `<SIZE-FOR-OSD>` must match the size of the local volumes created above.
> Therefore, we need to apply the value of `100Gi`.

```yaml
---
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  manageNodes: false
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
  - count: 1
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: <SIZE-FOR-OSD>Gi
        storageClassName: ocs-localblock
        volumeMode: Block
    name: ocs-deviceset
    placement: {}
    portable: false
    replica: 3
    resources: {}
...
```

### Apply the Configuration

Now apply the storage cluster Yaml file

```shell
oc create -f storage-cluster.yaml
```

Wait for the storage cluster to be setup.

> IMPORTANT: It will take several minutes, especially in a workshop lab environment, for the Storage Cluster to be completely rolled out and be functional. Be patient!

Check the Pods. There are multiple pods being created. Pods for the 'noobaa' Object Gateway will be created last.

```shell
bash-3.2$ oc get pods
NAME                                                              READY   STATUS      RESTARTS   AGE
aws-s3-provisioner-86dbb9bf9b-nqm7d                               1/1     Running     0          71m
csi-cephfsplugin-8tjmf                                            3/3     Running     0          53m
csi-cephfsplugin-gl4r8                                            3/3     Running     0          53m
csi-cephfsplugin-hnbkq                                            3/3     Running     0          53m
csi-cephfsplugin-hz98b                                            3/3     Running     0          53m
csi-cephfsplugin-jrpxg                                            3/3     Running     0          53m
csi-cephfsplugin-mfjxp                                            3/3     Running     0          53m
csi-cephfsplugin-n7f8r                                            3/3     Running     0          53m
csi-cephfsplugin-provisioner-7dccf64b5b-8tkjc                     5/5     Running     0          53m
csi-cephfsplugin-provisioner-7dccf64b5b-kqp68                     5/5     Running     0          53m
csi-cephfsplugin-r6khj                                            3/3     Running     0          53m
csi-cephfsplugin-wpmvt                                            3/3     Running     0          53m
csi-rbdplugin-5knbh                                               3/3     Running     0          53m
csi-rbdplugin-74jzx                                               3/3     Running     0          53m
csi-rbdplugin-7s7rt                                               3/3     Running     0          53m
csi-rbdplugin-dnbws                                               3/3     Running     0          53m
csi-rbdplugin-j54jg                                               3/3     Running     0          53m
csi-rbdplugin-mf9rr                                               3/3     Running     0          53m
csi-rbdplugin-n6xsn                                               3/3     Running     0          53m
csi-rbdplugin-provisioner-7cdfdc587-4sj79                         5/5     Running     0          53m
csi-rbdplugin-provisioner-7cdfdc587-s76td                         5/5     Running     0          53m
csi-rbdplugin-qrzp4                                               3/3     Running     0          53m
csi-rbdplugin-t7wxz                                               3/3     Running     0          53m
lib-bucket-provisioner-6b7fbd7bc4-4cfxv                           1/1     Running     0          71m
noobaa-core-0                                                     1/1     Running     0          47m
noobaa-db-0                                                       1/1     Running     0          47m
noobaa-default-backing-store-noobaa-noobaa-0                      1/1     Running     0          4m2s
noobaa-endpoint-78595688c4-9s5cb                                  1/1     Running     0          44m
noobaa-operator-74b886b9bf-b8m5s                                  1/1     Running     0          6m19s
ocs-operator-84ddb89994-7pc5f                                     1/1     Running     0          14m
rook-ceph-crashcollector-compute-0-6455866c78-xl2tr               1/1     Running     0          48m
rook-ceph-crashcollector-compute-1-b57f47fb5-ls55v                1/1     Running     0          48m
rook-ceph-crashcollector-compute-2-574c564668-zvdz5               1/1     Running     0          49m
rook-ceph-drain-canary-compute-0-796f8dcfcc-rdqr6                 1/1     Running     0          47m
rook-ceph-drain-canary-compute-1-6cfbb7fb88-qkql2                 1/1     Running     0          47m
rook-ceph-drain-canary-compute-2-f864ff67f-c8pxs                  1/1     Running     0          47m
rook-ceph-mds-ocs-storagecluster-cephfilesystem-a-bff5d7c6s4jqr   1/1     Running     0          46m
rook-ceph-mds-ocs-storagecluster-cephfilesystem-b-f84d6bc86znch   1/1     Running     0          46m
rook-ceph-mgr-a-6496bf7fc5-6k5qg                                  1/1     Running     0          48m
rook-ceph-mon-a-dd965dcf9-8cw46                                   1/1     Running     0          49m
rook-ceph-mon-b-6487bdf744-nxlkc                                  1/1     Running     0          48m
rook-ceph-mon-c-698f456cb-llpts                                   1/1     Running     0          48m
rook-ceph-operator-8479d65fc7-q5tds                               1/1     Running     0          71m
rook-ceph-osd-0-c877d7bcf-zkz2m                                   1/1     Running     0          47m
rook-ceph-osd-1-5c96f654d8-fjznr                                  1/1     Running     0          47m
rook-ceph-osd-2-767c8987b9-j7s49                                  1/1     Running     0          47m
rook-ceph-osd-prepare-ocs-deviceset-0-0-ljdsd-k8dpb               0/1     Completed   0          47m
rook-ceph-osd-prepare-ocs-deviceset-1-0-rhvpb-84nz4               0/1     Completed   0          47m
rook-ceph-osd-prepare-ocs-deviceset-2-0-kcxg2-stpsg               0/1     Completed   0          47m
```

Check the rollout state of the ClusterServiceVersion for OCS

```shell
oc get csvs -n openshift-storage
```

and look for status of 'ocs-operator.v4.y.z'

## Troubleshooting

### Noobaa Backing Store does not get created

The Operator-based rollout of OCS will install all components and as well create a default backing store for Noobaa.

When developing this module, it happened that this backing store was not created, likely due to issues with too high load (overload) of the lab server.

To fix the issue:

Using WebUI, Create a Noobaa backing store named "noobaa-default-backing-store" under 

* Namespace: openshift-storage
* Installed Operators > OpenShift Container Storage 4.y.z > Storage Cluster > 'ocs-storagecluster' > Resources > 'noobaa' > Resources
  Select Create New ... > "Backing Storage"
  * Name: noobaa-default-backing-store
  * Provider: PVC
  * Number of volumes: 1
  * Size: 16GiB (16GiB is the minimum size)
  * StorageClass: ocs-storagecluster-ceph-rbd

## Extend OCS by Additional Volumes

Documentation Link: [](https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.5/html-single/managing_openshift_container_storage/index#scaling-up-storage-by-adding-capacity-to-your-openshift-container-storage-nodes-using-local-storage-devices_rhocs)

### Add additional Disks to the Nodes

We already added an additional to each of the storage nodes when preparing the infrastructure for this lab. In a real-world scenario you would need to:

* prepare the disks
* put node into maintenance (cordon and drain the node)
* shutdown the node
* add disk to node (physically or on VM level)
* restart the node
* put node out of maintenance mode (uncordon node)

This would to be followed for each storage node one after another.

> > TODO - check if there are any special procedures to get OCS nodes into maintenance

### Add additional Disks using Local Storage Operator

Using Local Storage Operator setup, add the additional disks to the Local Storage Operator.

Update 'localvolume-ocs.yaml' and add the '/dev/vdc' disk we prepared.

```yaml
---
apiVersion: local.storage.openshift.io/v1
kind: LocalVolume
metadata:
  name: ocs-localblockvolume
  namespace: local-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: cluster.ocs.openshift.io/openshift-storage
        operator: In
        values:
        - ""
  storageClassDevices:
  - storageClassName: ocs-localblock
    volumeMode: Block
    devicePaths:
          - /dev/vdb
          - /dev/vdc
...
```

Apply the file using 'oc apply':

```shell
oc apply -f localvolume-ocs.yaml
```

and wait for the pods managing the disks to appear. 2 Pods per device will be available.

```shell
oc get pods -n local-storage
```

Run as well the following commands:

```shell
oc get pv
oc get sc
```

There should be a StorageClass 'ocs-localblock' and 3 PVs 'local-pv-....' for that StorageClass being created.

### Extend the OCS Storage Cluster

#### Extend with same-sized storage

If the added disks are same size as the original disks used for the storage cluster, simple incrase the 'StorageCluster.spec.storageDeviceSet.count' to '2' in the existing storage cluster definition and apply the changes. 

Example (not the 'count: 2' line):

```yaml
---
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  manageNodes: false
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
  - count: 2
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: <SIZE-FOR-OSD>Gi
        storageClassName: ocs-localblock
        volumeMode: Block
    name: ocs-deviceset
    placement: {}
    portable: false
    replica: 3
    resources: {}
...
```

#### Extend with different-sized storage

In case the storage size of the new disks is different, use another approach.
Extend existing 'storage-cluster.yaml':

* add another 'storageDeviceSet' similar to the existing
* give the device set a new name (here: 'ocs-deviceset2')
* define the storage size definition of the new disks
* keep storage class name

```yaml
---
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  manageNodes: false
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
  - count: 1
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 100Gi
        storageClassName: ocs-localblock
        volumeMode: Block
    name: ocs-deviceset
    placement: {}
    portable: false
    replica: 3
    resources: {}
  - count: 1
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 50Gi
        storageClassName: ocs-localblock
        volumeMode: Block
    name: ocs-deviceset2
    placement: {}
    portable: false
    replica: 3
    resources: {}
...
```

Now apply the storage cluster Yaml file

```
[root@bastion ~]# oc replace -f storage-cluster.yaml
```

Wait for OCS Operator to consume the changed configuration and rollout the extended storage.
