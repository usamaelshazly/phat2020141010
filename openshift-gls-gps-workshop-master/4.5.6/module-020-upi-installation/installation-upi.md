# Module01:

# Installation

We will define all of our initial virtual machines for the cluster with the following commands on the Hypervisor host:

### Bootstrap Virtual Machine:

```
[root@hypervisor ~]# virt-install -n bootstrap.hX.rhaw.io --description "Bootstrap Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:e1:78:8a
```

### Master01 Virtual Machine:

```
[root@hypervisor ~]# virt-install -n master01.hX.rhaw.io --description "Master01 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:f1:86:29
```

### Master02 Virtual Machine:

```
[root@hypervisor ~]# virt-install -n master02.hX.rhaw.io --description "Master02 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:af:63:f3
```

### Master03 Virtual Machine:

```
[root@hypervisor ~]# virt-install -n master03.hX.rhaw.io --description "Master03 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:a9:98:dd
```

### node01 Virtual Machine:

```
[root@hypervisor ~]# virt-install -n node01.hX.rhaw.io --description "node01 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:9f:95:87
```

### node02 Virtual Machine:

```
[root@hypervisor ~]# virt-install -n node02.hX.rhaw.io --description "node02 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk pool=ocp_images,bus=virtio,size=50 --graphics vnc --pxe --network network=ocp4-network,mac=52:54:00:c4:8f:50
```

A good practice is to start bootstrap vm first. Then step by step all other machines. They will start and boot up and they will select the proper CoreOS Image and the ignition file automatically and install and reboot.

Sometimes it could happen, that after the first boot the machines are in a loop and always trying to boot again from pxe. This could happen it must not happen.

If it happens we need to do the following steps described below.

In our workshop the virtual machines are set to --noreboot. After the machines are powered up and the CoreOS installing is done it will not reboot. This is because all these nodes master, node and bootstrap are in an headless mode.

So after 10 - 15 minutes we need to power off all of these nodes:

First we need to list all vm's:

```
[root@h12 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 1     bastion.h12.rhaw.io           running
 -     bootstrap.hX.rhaw.io           shut off
 -     master01.hX.rhaw.io            shut off
 -     master02.hX.rhaw.io            shut off
 -     master03.hX.rhaw.io            shut off
 -     node01.hX.rhaw.io            shut off
 -     node02.hX.rhaw.io            shut off
 ```

Now we need to poweroff all running machines:

```
[root@hypervisor ~]# virsh destroy bootstrap.hX.rhaw.io ...
```

Our virtual machines are configured to boot always from pxe we now need to change this so that from now on they are booting from disc.

You can observe the installation process if you access the bootstrap node from your service machine with virsh command.

The first command gives us a list of boot devices and their order:

```
[root@hypervisor ~]# virsh dumpxml bootstrap.hX.rhaw.io | grep 'boot order'
```

`2` is our virtual harddisk and `1` our nic.

Now we just need to change 2 into 1 and 1 into 2.

```
vim /etc/libvirt/qemu/bootstrap.hX.rhaw.io.xml
```

Search for boot order and change it accordingly for all Openshift virtual machines.

After we made these changes we need to reload the xml files to libvirt. We are doing this with the command:

```
[root@hypervisor ~]# virsh define bootstrap.hX.rhaw.io.xml
```

Now we can power on all our virtual machines with the command:

```
[root@hypervisor ~]# virsh start --domain bootstrap.hX.rhaw.io
```

```
[root@hypervisor ~]# virsh start --domain master01.hX.rhaw.io
```

```
[root@hypervisor ~]# virsh start --domain master02.hX.rhaw.io
```

```
[root@hypervisor ~]# virsh start --domain master03.hX.rhaw.io
```

```
[root@hypervisor ~]# virsh start --domain node01.hX.rhaw.io
```

```
[root@hypervisor ~]# virsh start --domain node02.hX.rhaw.io
```

You can observe the installation process if you access the bootstrap node from your service machine with the command:

```
[root@bastion ~]# ssh core@bootstrap.ocp4.hX.rhaw.io
```

After done this there is during the installation process a way of executing a journalctl command to observe this process.

To check the cluster is up and running type in the following command:

```
[root@bastion ~]# export KUBECONFIG=/root/ocp4/auth/kubeconfig
```

```
[root@bastion ~]# oc whoami
```

```
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    master,node   10m   v1.16.2
master02   Ready    master,node   10m   v1.16.2
master03   Ready    master,node   10m   v1.16.2
node01   Ready    node          10m   v1.16.2
node02   Ready    node          10m   v1.16.2
```

You should get an output of six machines in state Ready.

## Troubleshooting: Pending Certificates

When you add machines to a cluster, two pending certificates signing request (CSRs) are generated for each machine that you added. You must verify that these CSRs are approved or, if necessary, approve them yourself.

```
[root@bastion ~]# oc get nodes
NAME      STATUS    ROLES           AGE   VERSION
master01  Ready     master,node   10m   v1.16.2
master02  Ready     master,node   10m   v1.16.2
master03  Ready     master,node   10m   v1.16.2
node01  NotReady  node          10m   v1.16.2
node02  NotReady  node          10m   v1.16.2
```

The output lists all of the machines that we created.

Now we need to review the pending certificate signing requests (CSRs) and ensure that the you see a client and server request with `Pending` or `Approved` status for each machine that you added to the cluster:

```
[root@bastion ~]# oc get csr
```

```
NAME        AGE     REQUESTOR                                                                   CONDITION
csr-8b2br   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-8vnps   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-bfd72   5m26s   system:node:ip-10-0-50-126.us-east-2.compute.internal                       Pending
csr-c57lv   5m26s   system:node:ip-10-0-95-157.us-east-2.compute.internal                       Pending
```

> |
> Because the CSRs rotate automatically, approve your CSRs within an hour of adding the machines to the cluster. If you do not approve them within an hour, the certificates will rotate, and more than two certificates will be present for each node. You must approve all of these certificates. After you approve the initial CSRs, the subsequent node client CSRs are automatically approved by the cluster kube-controller-manager. You must implement a method of automatically approving the kubelet serving certificate requests.

Now we need to approve pending certificates:

```
[root@bastion ~]# oc adm certificate approve csr-bfd72
```

Hint:
To approve all pending certificates run the folloing command:

```
[root@bastion ~]# oc get csr -o name | xargs oc adm certificate approve
```

After that we can check the csr status again and validate that they are all "Approved,Issued":

```
[root@bastion ~]# oc get csr
```

## Intermediate Image Registry Storage Configuration

``This part was pushed into the registry chapter!``

## Completing installation on User Provisioned Infrastructure:

After we complete the operator configuration, you can finish installing the cluster on infrastructure that you provide.

We need to confirm that all components are up and running.

```
[root@bastion ~]# watch -n5 oc get clusteroperators
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.5.6     True        False         False      4m41s
cloud-credential                           4.5.6     True        False         False      25m
cluster-autoscaler                         4.5.6     True        False         False      15m
console                                    4.5.6     True        False         False      11m
dns                                        4.5.6     True        False         False      19m
image-registry                             4.5.6     True        False         False      16m
ingress                                    4.5.6     True        False         False      15m
insights                                   4.5.6     True        False         False      21m
kube-apiserver                             4.5.6     True        False         False      17m
kube-controller-manager                    4.5.6     True        False         False      17m
kube-scheduler                             4.5.6     True        False         False      18m
machine-api                                4.5.6     True        False         False      20m
machine-config                             4.5.6     True        False         False      19m
marketplace                                4.5.6     True        False         False      15m
monitoring                                 4.5.6     True        False         False      10m
network                                    4.5.6     True        False         False      21m
node-tuning                                4.5.6     True        False         False      15m
openshift-apiserver                        4.5.6     True        False         False      15m
openshift-controller-manager               4.5.6     True        False         False      18m
openshift-samples                          4.5.6     True        False         False      14m
operator-lifecycle-manager                 4.5.6     True        False         False      20m
operator-lifecycle-manager-catalog         4.5.6     True        False         False      20m
operator-lifecycle-manager-packageserver   4.5.6     True        False         False      15m
service-ca                                 4.5.6     True        False         False      21m
service-catalog-apiserver                  4.5.6     True        False         False      16m
service-catalog-controller-manager         4.5.6     True        False         False      16m
storage                                    4.5.6     True        False         False      16m
```

  When all of the cluster Operators are available (the kube-apiserver operator is last in state PROGRESSING True and takes roughly 15min to finish), we can complete the installation.

> The Ignition config files that the installation program generates contain certificates that expire after 24 hours. You must keep the cluster running for 24 hours in a non-degraded state to ensure that the first certificate rotation has finished.
