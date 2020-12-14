## Adding new Node to Cluster

An important task for operations is to adding new nodes to the cluster. The reason could me more workload on the cluster. Nodes for specific reasons (infrastructure nodes), nodes for specific purpouse (nodes for specific customers)

The procedure is quite similar to the installation process.

In our environment first we need to create a new virtual node:

node 03

```
virt-install -n node03.hX.rhaw.io --description "node03 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk path=/mnt/ocp_images/node03.qcow2,bus=virtio,size=50 --graphics none --pxe --network network=ocp4-network,mac=52:54:00:fe:e5:e3
```

After we have done this, the node will power on and will be pxe booted. After that it will fetch the CoreOS image and the related ignition file and will start the installation.

The node will stop after installation and we need to start the node with the command:

```
virsh start --domain node03.hX.rhaw.io
```

Now we need to do the same steps as in the cluster installation. This steps are:

```
[root@hypervisor ~]# virsh list --all
```

```
Id    Name                           Status
----------------------------------------------------
 15    workstation.hX.rhaw.io    laufend
 18    bastion.hX.rhaw.io       laufend
 20    bootstrap.hX.rhaw.io      laufend
 22    master01.hX.rhaw.io       laufend
 24    master02.hX.rhaw.io       laufend
 27    master03.hX.rhaw.io       laufend
 29    node01.hX.rhaw.io       laufend
 30    node02.hX.rhaw.io       laufend
```

After the node03 node has been installed we can't see it in the list of nodes:

```
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES           AGE    VERSION
master01   Ready    master,node   24h    v1.16.2
master02   Ready    master,node   24h    v1.16.2
master03   Ready    master,node   24h    v1.16.2
node01   Ready    node          24h    v1.16.2
node02   Ready    node          24h    v1.16.2
```

When we look at the csr's in our cluster we can see that some are in pending mode. This is because of our new node. 

```
[root@bastion ~]# oc get csr
NAME        AGE     REQUESTOR                                                                   CONDITION
csr-66q24   48m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-9p5kh   2m47s   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-fl4f9   63m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-mbr5t   18m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-wkrsp   33m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
```

we have to approve all pending certificates:

```
oc get csr -o name | xargs oc adm certificate approve
```

We need a secound round:

```
[root@bastion ~]# oc get csr
```

```
NAME        AGE     REQUESTOR                                                                   CONDITION
csr-72t87   4s      system:node:node03                                                        Pending
csr-f6tsc   20m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-lrc8f   5m19s   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
```

And a last approvel:

```
oc adm certificate approve CERTIFICATE
```

After a couple of minutes the machine should be up and running and part of the cluster as a third node node:

```
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES           AGE    VERSION
master01   Ready    master,node   24h    v1.16.2
master02   Ready    master,node   24h    v1.16.2
master03   Ready    master,node   24h    v1.16.2
node01   Ready    node          24h    v1.16.2
node02   Ready    node          24h    v1.16.2
node03   Ready    node          113s   v1.16.2
```

For the next Step in our Workshop we will add another node to the cluster node04.hX.rhaw.io:

```
virt-install -n node04.hX.rhaw.io --description "node04 Machine for Openshift 4 Cluster" --os-type=Linux --os-variant=rhel7 --ram=8192 --vcpus=4 --noreboot --disk path=/mnt/ocp_images/node04.qcow2,bus=virtio,size=50 --graphics none --pxe --network network=ocp4-network,mac=52:54:00:f1:79:58
```

After that we will just follow the steps we did with node03.hX.rhaw.io.

## Adding new Node to Cluster after 24+ hours

> After 24 hours the cluster rotates the first time the certificates! If you want to add a new node after this 24 hours we have to use the new certificate!

When installing OCP4 clusters a bootstrap certificate is created and is used on the master nodes.
Because certificates can not be revoked, this certificate is made with a short expiry time and 24 hours after cluster installation, it can not be used again.

An attempt to add a new node ends up with a failure due to certificate signed by unknown authority!

In general, one has to deploy a new node in the same way as during the installation process but using updated ignition file.

The directory with manifest and Ignition config files that were used for the installation is required.

Below are MANUAL and AUTOMATED approach, in both cases create some temporary directory (IMPORTANT) where you put node.ign file before you start.

##### MANUAL

Retrieve the certificate from the api-int.api-int.<cluster.domain>:22623 and save in a temporary file (for example api-int.pem)

```
openssl s_client -connect api-int.${domain}:22623 -showcerts
```

> NOTE: the certificate is a block of text starting with "BEGIN CERTIFICATE" and ending at "END CERTIFICATE"

Encode the certificate using base64 with "--wrap=0" option

```
base64 --wrap=0 ./api-int.pem > ./api-int.base64.pem
```

Just in case make a backup of node.ign and then replace ignition.security.tls.certificateAuthorities[0].source with a new value obtained in the previous step

```
cp ./node.ign ./node.ign.backup
```

put the content of api-int.base64 file to the node.ign file and place it as below instead of <VALUE>

```json
{
  "ignition": {
    "config": {
      "append": [
        {
          "source": "https://api-int.<cluster.domain>:22623/config/metal-node",
          "verification": {}
        }
      ]
    },
    "security": {
      "tls": {
        "certificateAuthorities": [
          {
            "source": "data:text/plain;charset=utf-8;base64,<VALUE>",
            "verification": {}
          }
        ]
      }
    },
    "timeouts": {},
    "version": "2.2.0"
  },
  "networkd": {},
  "passwd": {},
  "storage": {},
  "systemd": {}
}
```

Upload the updated node.ign file to the HTTP server you were using for your OCP cluster deployment and start the node machine.

##### AUTOMATED

Replace api-int.api-int.<cluster.domain> with your FQDN only and run below commands

```
echo "q" | openssl s_client -connect api-int.<cluster.domain>:22623  -showcerts | awk '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/' | base64 --wrap=0 | tee ./api-int.base64 && \
sed --regexp-extended --in-place=.backup "s%base64,[^,]+%base64,$(cat ./api-int.base64)\"%" ./node.ign
```

Check is the file changed, if yes, then upload the updated node.ign file to the HTTP server you were using for your OCP cluster deployment and start the node machine.