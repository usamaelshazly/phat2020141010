# Module 04:  NFS Storage

OpenShift Container Platform requires storage for certain internal components. These are

- Registry
- Monitoring (Prometheus and Prometheus Alert Manager)
- Aggregates Logging (ElasticSearch component of OpenShift's aggregated logging component stack)

In addition, applications may require and request persistent storage via Persistent Volume Claims (PVCs).
For a general introduction on persistent storage see the [Persistent storage overview](https://docs.openshift.com/container-platform/4.2/storage/understanding-persistent-storage.html) chapter of the product documentation.

Overall, persistent storage needs to be carefully selected based upon the requirements of the respective components using the storage. These requirements include

- type of storage (file, block or object storage)
- amount of storage
- non-functional aspects such as performance and capacity requirements
- etc.

## NFS Storage

In this course we will use NFS storage to satisfy the persistent storage requirements of the OpenShift internal components and demonstrate storage capabilities for applications.

> Note:
> 
> NFS storage is **NOT** a generic approach and 'fits every scenario' solution. NFS storage has deficiencies for certain scenarios.
> 
> We will use for simplicity in this course NFS storage.

### Setup of a local NFS Server

We will use our bastion machine to setup a NFS server for use with our cluster. This acts for demonstration purposes. In a real-world environment dedicated storage backend servers would be used.

For a basic NFS server setup, execute the following commands on the bastion VM as root:

```
[root@bastion ~]# rpm -q nfs-utils container-selinux
```

```
[root@bastion ~]# systemctl enable --now nfs-server rpcbind
```

```
[root@bastion ~]# firewall-cmd --add-service=nfs --permanent
```

```
[root@bastion ~]# firewall-cmd --add-service={nfs3,mountd,rpc-bind} --permanent
```

```
[root@bastion ~]# firewall-cmd --reload
```

```
[root@bastion ~]# setsebool -P nfs_export_all_rw 1
```

We now create dedicated export directories for the storage of the OpenShift internal components and a series of directories for application PVs, and set permissions. Overall, we need shares for

- OpenShift Registry 
- OpenShift Monitoring (2 Shares for Prometheus and 3 Shares for Prometheus AlertManager)
- OpenShift Aggregated Logging
- OpenShift Metering (TODO)

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/registry
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/monitoring/p-0
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/monitoring/p-1
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/monitoring/a-0
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/monitoring/a-1
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/monitoring/a-2
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/logging/es-1
```

```
[root@bastion ~]# mkdir -p /data/nfs/sys-vols/logging/es-2
```

```
[root@bastion ~]# mkdir -p /data/nfs/user-vols/pv{1..50}
```

```
[root@bastion ~]# chown -R nobody.nobody /data/nfs
```

```
[root@bastion ~]# chmod -R 777 /data/nfs
```

Finally, create the exports and restart the NFS server service:

```
[root@bastion ~]# echo /data/nfs/sys-vols/registry *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-registry.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/monitoring/p-0 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-monitoring-pr-0.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/monitoring/p-1 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-monitoring-pr-1.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/monitoring/a-0 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-monitoring-am-0.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/monitoring/a-1 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-monitoring-am-1.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/monitoring/a-2 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-monitoring-am-2.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/logging/es-1 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-elastic-1.exports
```

```
[root@bastion ~]# echo /data/nfs/sys-vols/logging/es-2 *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0) >> /etc/exports.d/openshift-sysvols-elastic-2.exports
```

```
[root@bastion ~]# for pvnum in {1..50} ; do
 echo /data/nfs/user-vols/pv${pvnum} *(rw,root_squash) >> /etc/exports.d/openshift-uservols.exports
done
```

```
[root@bastion ~]# exportfs -rav
```

```
[root@bastion ~]# systemctl reload-or-restart nfs-server
```

### Testing access to the NFS Server

In this module, we will act as system:admin user and create PV and PVC,
just to make sure that our NFS setup principally works.

Create a file 'nfs-pv.yaml' on the bastion machine having the following content:

```
mkdir -p /root/openshift/nfs-configuration/
```

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/nfs-pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/user-vols/pv50
    server: bastion.hX.rhaw.io
    readOnly: false
```

Create a file 'nfs-pvc.yaml' on the bastion machine having the following content:

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/nfs-pvc.yaml
```

```yam
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  resources:
     requests:
       storage: 1Gi
```

Now, create the PV:

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/nfs-pv.yaml
```

```
[root@bastion ~]# oc get pv
```

```
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
nfs-pv 1Gi RWX Retain Available
```

We will now create an example project and import the PVC. The PVC should get bound to the existing PV:

```
[root@bastion ~]# oc new-project pvctest
```

```
[root@bastion ~]# oc status
```

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/nfs-pvc.yaml
```

```
[root@bastion ~]# oc get pvc
```

```
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
test-pvc Bound nfs-pv 1Gi RWX 20h
```

```
[root@bastion ~]# oc get pv
```

```
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
nfs-pv 1Gi RWX Retain Bound pvctest/test-pvc 20h
```

TODO: clean-up the above created ressources

## Configuring persistent NFS storage for the Registry

Create a file 'registry-pv.yaml' having the following content and load it into the cluster using `oc create -f registry-pv.yaml`.

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/registry-pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: registry-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/sys-vols/registry
    server: bastion.hX.rhaw.io
    readOnly: false
```

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/registry-pv.yaml
```

The storage for the registry is defined in the config of the registry operator.

Remove any existing storage definition. Remember we have modified during installation the registry to use 'emptyDir', i.e., ephemeral storage. The following command removes the storage definition:

```
[root@bastion ~]# oc patch configs.imageregistry.operator.openshift.io cluster --type json --patch '[{ "op": "remove", "path": "/spec/storage" }]'
```

Now add a persistent storage with a PVC storage definition.

```
[root@bastion ~]# oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"pvc":{"claim":""}}}}'
```

Watch the output of

```
[root@bastion ~]# oc get clusteroperator image-registry
```

Once 'progressing' moves to 'false', the operator has updated the rolled-out registry, and a registry PVC has been created. That PVC should have been bound to the above created PV.

```
[root@bastion ~]# oc get pv
```

```
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
registry-pv 100Gi RWX Retain Bound openshift-image-registry/image-registry-storage 87m
```

## Configuring Persistent Storage for Monitoring

> TODO

See [Recommended configurable storage technology](https://docs.openshift.com/container-platform/4.2/scalability_and_performance/optimizing-storage.html#recommended-configurable-storage-technology_persistent-storage) for general storage type recommendations.

In this workshop we use NFS file storage for monitoring though this is not the recommended storage type. Recommended is block storage.

See as well [Monitoring - Configuring persistent storage](https://docs.openshift.com/container-platform/4.2/monitoring/cluster-monitoring/configuring-the-monitoring-stack.html#configuring-persistent-storage) for additional details on the configuration of persistent storage for Monitoring.

Check whether the cluster-monitoring-config ConfigMap object exists:

```
[root@bastion ~]# oc -n openshift-monitoring get configmap cluster-monitoring-config
```

If it does not exist, create it:

```
[root@bastion ~]# cd /root/openshift/nfs-configuration/
```

```
[root@bastion ~]# oc -n openshift-monitoring create configmap cluster-monitoring-config
```

Edit the created config map using `oc -n openshift-monitoring edit configmap cluster-monitoring-config ` and put the following data section

```
[root@bastion ~]# oc -n openshift-monitoring edit configmap cluster-monitoring-config
```

```yaml
data:
  config.yaml: |
    prometheusK8s:
      volumeClaimTemplate:
        metadata:
          name: prometheus-pvc
        spec:
          resources:
            requests:
              storage: 10Gi
          selector:
            matchLabels:
              infrapvc: "prometheus"
    alertmanagerMain:
      volumeClaimTemplate:
        metadata:
          name: alertmanager-pvc
        spec:
          resources:
            requests:
              storage: 2Gi
          selector:
            matchLabels:
              infrapvc: "alertmanager"
```

Save the changes to the ConfigMap. The operator will now apply the changed configuration and rollout new pods. You can use e.g. `oc get pods` to look for the new pods getting created.

We now have to ensure, Persistent Volumes (PVs) are ready to be claimed by the Persistent Volume Claim (PVC) created by the configuration change. One PV for each replica is required. By default, Prometheus uses two replicas and Alertmanager uses three replicas, hence we need five PVs to support the entire monitoring stack.

Check the PVCs created by the configuration change.

```
[root@bastion ~]# oc get pvc -n openshift-monitoring
```

```
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
alertmanager-pvc-alertmanager-main-0 Pending 11s
alertmanager-pvc-alertmanager-main-1 Pending 11s
alertmanager-pvc-alertmanager-main-2 Pending 11s
prometheus-pvc-prometheus-k8s-0 Pending 7s
prometheus-pvc-prometheus-k8s-1 Pending 7s
```

There are 5 PVCs created which are yet missing the required PVs. We now prepare these PVs as NFS-backed PVs.

Inspect one of the PVCs:

```
[root@bastion ~]# oc get pvc prometheus-pvc-prometheus-k8s-0 -o yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 creationTimestamp: "2020-01-09T16:00:01Z"
 finalizers:
 - kubernetes.io/pvc-protection
 labels:
 app: prometheus
 prometheus: k8s
 name: prometheus-pvc-prometheus-k8s-0
 namespace: openshift-monitoring
 resourceVersion: "741271"
 selfLink: /api/v1/namespaces/openshift-monitoring/persistentvolumeclaims/prometheus-pvc-prometheus-k8s-0
 uid: 18001914-32f9-11ea-9b39-525400af63f3
spec:
 accessModes:
 - ReadWriteOnce
 resources:
 requests:
 storage: 10Gi
 selector:
 matchLabels:
 infrapvc: prometheus
 volumeMode: Filesystem
status:
 phase: Pending
```

You can see the PVCs use 'matchLabel' selectors, hence we have to prepare the PVs to have respective labels as well.

Create the PV YAML definitions.

Create two YAML files for the Prometheus PVs as follows, pointing to the NFS server and respective shares '.../monitoring/p-0' and '.../monitoring/p-1' as follows:

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/m-pv0.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: monitoring-pv-0
  labels:
    infrapvc: prometheus
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/sys-vols/monitoring/p-0
    server: bastion.hX.rhaw.io
    readOnly: false
```

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/m-pv1.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: monitoring-pv-1
  labels:
    infrapvc: prometheus
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
 persistentVolumeReclaimPolicy: Retain
 nfs:
   path: /data/nfs/sys-vols/monitoring/p-1
   server: bastion.hX.rhaw.io
   readOnly: false
```

Create three YAML files for the AlertManager PVs as follows, pointing to the NFS server and respective shares '.../monitoring/a-0' to '.../monitoring/a-2' as follows:

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/alert-pv0.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: monitoring-alertmgr-pv-0
  labels:
    infrapvc: alertmanager
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/sys-vols/monitoring/a-0
    server: bastion.hX.rhaw.io
    readOnly: false
```

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/alert-pv1.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: monitoring-alertmgr-pv-1
  labels:
    infrapvc: alertmanager
spec:
  capacity:
    storage: 10Gi
 accessModes:
   - ReadWriteOnce
 persistentVolumeReclaimPolicy: Retain
 nfs:
   path: /data/nfs/sys-vols/monitoring/a-1
   server: bastion.hX.rhaw.io
   readOnly: false
```

```
[root@bastion ~]# vim /root/openshift/nfs-configuration/alert-pv2.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: monitoring-alertmgr-pv-2
 labels:
 infrapvc: alertmanager
spec:
 capacity:
 storage: 10Gi
 accessModes:
 - ReadWriteOnce
 persistentVolumeReclaimPolicy: Retain
 nfs:
 path: /data/nfs/sys-vols/monitoring/a-2
 server: bastion.hX.rhaw.io
 readOnly: false
```

Remember to not only adjust the path on the NFS server but as well the PV name in the metadata section.

> Observe that the PVs for Prometheus and Alertmanager use label 'infrapvc' with respective values mathing the label selectors in the PVs.

Now import the PV using `oc create -f filename.yaml` for each of the PV files created.

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/alert-pv0.yaml
```

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/alert-pv1.yaml
```

```
[root@bastion ~]# oc create -f /root/openshift/nfs-configuration/alert-pv2.yaml
```

Check the PVCs (or PVs respectively) in the system to ensure the PVCs get bound.

```
[root@bastion ~]# oc get pvc -n openshift-monitoring
```

```
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
alertmanager-pvc-alertmanager-main-0 Bound monitoring-alertmgr-pv-0 10Gi RWO 5h16m
alertmanager-pvc-alertmanager-main-1 Bound monitoring-alertmgr-pv-1 10Gi RWO 5h16m
alertmanager-pvc-alertmanager-main-2 Bound monitoring-alertmgr-pv-2 10Gi RWO 5h16m
prometheus-pvc-prometheus-k8s-0 Bound monitoring-pv-0 10Gi RWO 5h16m
prometheus-pvc-prometheus-k8s-1 Bound monitoring-pv-1 10Gi RWO 5h16m
```

## Configuring Persistent Storage for Logging

> TODO

To add persistent storage for Logging, i.e., to the ElasticSearch component of OpenShift Cluster Logging, we assume cluster logging is already installed with ephemaral storage as described earlier in this workshop.

Overall, we now could add the storage definition to the 'ClusterLogging' CRD created earlier, replacing the ephemeral storage definition.

To demonstrate the rollout of logging including persistent storage, we simply delete the installed cluster logging components and re-rollout using a new definition including the persistent storage.

### Remove installed Cluster Logging instance

Reference: [Product Documentation - Uninstalling Cluster Logging](https://docs.openshift.com/container-platform/4.2/logging/cluster-logging-uninstall.html)

Use the following command to remove everything generated during the deployment.

```
[root@bastion ~]# oc delete clusterlogging instance -n openshift-logging
```

This will remove all pods, deplyoments, bastion, routes. Watch the removal ov objects using `oc get all -n openshift-logging`. Only the cluster operator will remain:

```
[root@bastion ~]# oc get all
```

```
NAME                                            READY   STATUS    RESTARTS   AGE
pod/cluster-logging-operator-5cb9bf8c7f-pfg6t   1/1     Running   1          2d4h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator   1/1     1            1           2d4h

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-5cb9bf8c7f   1         1         1       2d4h
```

There should not by any PVCs in namespace 'openshift-logging' yet (we did not create any so far), but for sake of completeness, delete the PVCs as well:

```
[root@bastion ~]# oc delete pvc --all -n openshift-logging
```

### Install Cluster Logging using Persistent Storage

We now install Cluster Logging again using the known steps, however we add persistent storage configuration to the CRD.

You can re-use the existing 'cust-resource-def-logging.yaml' file of the CRD and simply edit it. As previously, we rollout only 2 ElasticSearch replicas and configure ElasticSearch for a minimal memory requests (8GB rather than the default 16 GB) due to the available resources in our workshop environment.

```yaml
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance" 
  namespace: "openshift-logging"
spec:
  managementState: "Managed"  
  logStore:
    type: "elasticsearch"  
    elasticsearch:
      nodeCount: 2
      storage:
        size: 20G
        storageClassName: ""
      redundancyPolicy: "SingleRedundancy"
      resources:
        limits:
          memory: "8Gi"
        requests:
          cpu: "1"
          memory: "8Gi"
  visualization:
    type: "kibana"  
    kibana:
      replicas: 1
  curation:
    type: "curator"  
    curator:
      schedule: "30 3 * * *"
  collection:
    logs:
      type: "fluentd"  
      fluentd: {}
```

> Note the only difference to the installation before is the change in the 'spec.logStore.elasticsearch.storage' definition.

Wait a few moments and look for the Operator to rollout route, bastion, deployments, and pods.

Note that now PVCs got created:

```
[root@bastion ~]# oc get pvc
NAME                                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
elasticsearch-elasticsearch-cdm-y9htwcau-1   Pending                                                     2m26s
elasticsearch-elasticsearch-cdm-y9htwcau-2   Pending                                                     2m26s
```

Create 2 PV definitions for the created PVCs and loading them using `oc create -f es-pv-1.yaml` (adjusting the following example of the first PV for the 2nd PV):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elasticsearch-pv-1
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/sys-vols/logging/es-1
    server: bastion.hX.rhaw.io
    readOnly: false
```

After a few seconds, the PVCs should get bound:

```
[root@bastion ~]# oc get pvc
NAME                                         STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
elasticsearch-elasticsearch-cdm-y9htwcau-1   Bound    elasticsearch-pv-1   20Gi       RWO                           3m34s
elasticsearch-elasticsearch-cdm-y9htwcau-2   Bound    elasticsearch-pv-2   20Gi       RWO                           3m34s
```

## Configuring Persistent Storage for Metering
