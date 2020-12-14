# ETCD

## Explore etcd

#### etcdclt version

On an OpenShift 4.x cluster the version 3.3.17 is used

```
sh-4.2# etcdctl version 
etcdctl version: 3.3.17
API version: 3.3
```

And the ETCDCTL_API version 3 is used by default

```
sh-4.2# env|grep ETCDCTL_API
ETCDCTL_API=3
```

#### Set the etcdctl environment variables

> In OpenShift 4.x the certificates for the etcd servers are now located in the directory /etc/ssl/etcd/!

```
sh-4.2# ls -lisa /etc/ssl/etcd
total 44
 89429870 4 drwxr-xr-x. 2 root root 4096 Mar  3 10:33 .
142711695 0 drwxr-xr-x. 1 root root   18 Mar 30 06:22 ..
 89429871 4 -rw-r--r--. 1 root root 1135 Mar  3 10:32 ca.crt
 89429872 4 -rw-r--r--. 1 root root 1151 Mar  3 10:32 metric-ca.crt
 89429873 8 -rw-r--r--. 1 root root 6998 Mar  3 10:32 root-ca.crt
 89429879 4 -rw-r--r--. 1 root root 1537 Mar  3 09:13 system:etcd-metric:etcd-0.ocp4.h12.rhaw.io.crt
 89429878 4 -rw-------. 1 root root 1679 Mar  3 09:13 system:etcd-metric:etcd-0.ocp4.h12.rhaw.io.key
 89429877 4 -rw-r--r--. 1 root root 1367 Mar  3 09:13 system:etcd-peer:etcd-0.ocp4.h12.rhaw.io.crt
 89429876 4 -rw-------. 1 root root 1675 Mar  3 09:13 system:etcd-peer:etcd-0.ocp4.h12.rhaw.io.key
 89429875 4 -rw-r--r--. 1 root root 1537 Mar  3 09:13 system:etcd-server:etcd-0.ocp4.h12.rhaw.io.crt
 89429874 4 -rw-------. 1 root root 1675 Mar  3 09:13 system:etcd-server:etcd-0.ocp4.h12.rhaw.io.key
```

To avoid typing on each etcdctl command the global options --cacert, --cert and --key we'll set the environment variables like this

```
sh-4.2# export ETCDCTL_CACERT=/etc/ssl/etcd/ca.crt ETCDCTL_CERT=$(find /etc/ssl/ -name *peer*crt) ETCDCTL_KEY=$(find /etc/ssl/ -name *peer*key)
```

Same is for the global environment variable ETCDCTL_ENDPOINTS

```
sh-4.2# export ETCDCTL_ENDPOINTS=$(etcdctl member list|cut -d',' -f5|cut -d ' ' -f2|awk 'ORS=","'| head -c -1)
```

> If the etcdctl command expects the list of cluster members the global environment variable is used if the variable isn't set add the option --cluster

#### Get the etcd member list

MEMBER LIST prints the member details for all members associated with an etcd cluster.

```
sh-4.2# etcdctl member list -w table
+------------------+---------+----------------------+--------------------------------------+-----------------------------+
|        ID        | STATUS  |         NAME         |              PEER ADDRS              |        CLIENT ADDRS         |
+------------------+---------+----------------------+--------------------------------------+-----------------------------+
| 570db60db6086170 | started | etcd-member-master02 | https://etcd-1.ocp4.h12.rhaw.io:2380 | https://192.168.100.22:2379 |
| ac4f6a42ebd3adfb | started | etcd-member-master03 | https://etcd-2.ocp4.h12.rhaw.io:2380 | https://192.168.100.23:2379 |
| d241e2cf814e2ec1 | started | etcd-member-master01 | https://etcd-0.ocp4.h12.rhaw.io:2380 | https://192.168.100.21:2379 |
+------------------+---------+----------------------+--------------------------------------+-----------------------------+
```

#### ETCD endpoint status

ENDPOINT STATUS queries the status of each endpoint in the given endpoint list.

```
sh-4.2# etcdctl endpoint status --cluster -w table
+--------------------------------------+------------------+---------+---------+-----------+-----------+------------+
|               ENDPOINT               |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+--------------------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://etcd-0.ocp4.h12.rhaw.io:2379 | d241e2cf814e2ec1 |  3.3.17 |   69 MB |     false |         8 |     945547 |
| https://etcd-1.ocp4.h12.rhaw.io:2379 | 570db60db6086170 |  3.3.17 |   69 MB |      true |         8 |     945547 |
| https://etcd-2.ocp4.h12.rhaw.io:2379 | ac4f6a42ebd3adfb |  3.3.17 |   70 MB |     false |         8 |     945547 |
+--------------------------------------+------------------+---------+---------+-----------+-----------+------------+
```

#### ETCD endpoint health

ENDPOINT HEALTH checks the health of the list of endpoints with respect to cluster. An endpoint is unhealthy when it cannot participate in consensus with the rest of the cluster.
If an endpoint can participate in consensus, prints a message indicating the endpoint is healthy. If an endpoint fails to participate in consensus, prints a message indicating the endpoint is unhealthy.

```
sh-4.2# etcdctl endpoint health --cluster -w table
+--------------------------------------+--------+-------------+-------+
|               ENDPOINT               | HEALTH |    TOOK     | ERROR |
+--------------------------------------+--------+-------------+-------+
| https://etcd-2.ocp4.h12.rhaw.io:2379 |   true | 20.306241ms |       |
| https://etcd-0.ocp4.h12.rhaw.io:2379 |   true | 12.822589ms |       |
| https://etcd-1.ocp4.h12.rhaw.io:2379 |   true | 23.781412ms |       |
+--------------------------------------+--------+-------------+-------+
```

#### Get the current existing objects in the etcd database

GET gets the key or a range of keys

To see how many enties are available for each key run:

```
sh-4.2# etcdctl get / --prefix --keys-only | sed '/^$/d' | cut -d/ -f3 | sort | uniq -c | sort -rn         
    950 secrets
    311 configmaps
    261 serviceaccounts
    245 rolebindings
    232 pods
    221 images
    184 clusterroles
    159 clusterrolebindings
    123 templates
    120 bastion
     77 roles
     76 replicasets
     74 imagestreams
     74 apiextensions.k8s.io
     67 apiregistration.k8s.io
     66 monitoring.coreos.com
     61 network.openshift.io
     53 namespaces
     49 deployments
     43 config.openshift.io
     30 controllerrevisions
     15 machineconfiguration.openshift.io
     13 operator.openshift.io
     13 daemonsets
     13 cloudcredential.openshift.io
     10 operators.coreos.com
      8 security.openshift.io
      7 routes
      7 minions
      7 leases
      6 oauth
      3 masterleases
      3 console.openshift.io
      2 validatingwebhookconfigurations
      2 users
      2 useridentities
      2 statefulsets
      2 ranges
      2 priorityclasses
      2 events
      1 tuned.openshift.io
      1 samples.operator.openshift.io
      1 rangeallocations
      1 poddisruptionbudgets
      1 imageregistry.operator.openshift.io
```

### etcd alarms

List all alarms

```
sh-4.2# etcdctl alarm list
```

Disarms all alarms

```
sh-4.2# etcdctl alarm disarm
```

### compact etcd

COMPACTION discards all etcd event history prior to a given revision.

An etcd cluster needs periodic maintenance to remain reliable. Depending on an etcd application's needs, this maintenance can usually be automated and performed without downtime or significantly degraded performance.
All etcd maintenance manages storage resources consumed by the etcd keyspace. Failure to adequately control the keyspace size is guarded by storage space quotas; if an etcd member runs low on space, a quota will trigger cluster-wide alarms which will put the system into a limited-operation maintenance mode. To avoid running out of space for writes to the keyspace, the etcd keyspace history must be compacted. Storage space itself may be reclaimed by defragmenting etcd members. Finally, periodic snapshot backups of etcd member state makes it possible to recover any unintended logical data loss or corruption caused by operational error.

Since etcd keeps an exact history of its keyspace, this history should be periodically compacted to avoid performance degradation and eventual storage space exhaustion. Compacting the keyspace history drops all information about keys superseded prior to a given keyspace revision. The space used by these keys then becomes available for additional writes to the keyspace.
The keyspace can be compacted automatically with `etcd`'s time windowed history retention policy, or manually with `etcdctl`. The `etcdctl` method provides fine-grained control over the compacting process whereas automatic compacting fits applications that only need key history for some length of time.

```
sh-4.2# REVISION=$(etcdctl endpoint status --cluster --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9].*'| sort -rn| tail -1)
```

```
sh-4.2# etcdctl compaction \
--endpoints=https://etcd-0.ocp4.h12.rhaw.io:2379,https://etcd-1.ocp4.h12.rhaw.io:2379,https://etcd-2.ocp4.h12.rhaw.io:2379 \
$REVISION
```

### defrag etcd

DEFRAG defragments the backend database file for a set of given endpoints while etcd is running

After compacting the keyspace, the backend database may exhibit internal fragmentation. Any internal fragmentation is space that is free to use by the backend but still consumes storage space. Compacting old revisions internally fragments `etcd` by leaving gaps in backend database. Fragmented space is available for use by `etcd` but unavailable to the host filesystem. In other words, deleting application data does not reclaim the space on disk.

The process of defragmentation releases this storage space back to the file system. Defragmentation is issued on a per-member so that cluster-wide latency spikes may be avoided.

To defragment an etcd member, use the `etcdctl defrag` command:

```sh
sh-4.2# etcdctl defrag
Finished defragmenting etcd member[127.0.0.1:2379]
```

> **Note that defragmentation to a live member blocks the system from reading and writing data while rebuilding its states**.

> **Note that defragmentation request does not get replicated over cluster. That is, the request is only applied to the local node. Specify all members in --endpoints flag or define the global environment variable ETCDCTL_ENDPOINTS**

Run defragment operations for all endpoints in the cluster associated with the default endpoint:

```
sh-4.2# etcdctl defrag          
Finished defragmenting etcd member[https://etcd-0.ocp4.h12.rhaw.io:2379]
Finished defragmenting etcd member[https://etcd-1.ocp4.h12.rhaw.io:2379]
Finished defragmenting etcd member[https://etcd-2.ocp4.h12.rhaw.io:2379]
```

For each endpoints, prints a message indicating whether the endpoint was successfully defragmented.
If there are error messages rerun the defrag with only the failed machine mentioned in --endpoints

### create etcd snapshots

SNAPSHOT SAVE writes a point-in-time snapshot of the etcd backend database to a file.

Snapshotting the `etcd` cluster on a regular basis serves as a durable backup for an etcd keyspace. By taking periodic snapshots of an etcd member's backend database, an `etcd` cluster can be recovered to a point in time with a known good state.
A snapshot is taken with `etcdctl`:

```
sh-4.2# etcdctl snapshot save /var/lib/etcd/backup/etcd-$(date +%Y%m%d)/db
{"level":"warn","ts":"2020-02-13T14:22:29.278Z","caller":"clientv3/retry_interceptor.go:116","msg":"retry stream intercept"}
Snapshot saved at /var/lib/etcd/backup/etcd-20200213/db
```

### validate etcd snapshots

SNAPSHOT STATUS lists information about a given backend database snapshot file.

```
sh-4.2# etcdctl snapshot status /var/lib/etcd/backup/etcd-$(date +%Y%m%d)/db -w table
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 196aa56f |   768941 |       5989 |      70 MB |
+----------+----------+------------+------------+
```

### ETCD performance check

There are four different load profiles available (s,m,l and xl)

| Profile | write limit | clients |
| ------- | ----------- | ------- |
| s       | 150         | 50      |
| m       | 1000        | 200     |
| l       | 8000        | 500     |
| xl      | 15000       | 1000    |

```
sh-4.2# etcdctl check perf --command-timeout=60s --load="s" --prefix="/etcdctl-check-perf/"
 60 / 60 Boooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00%1m0s
PASS: Throughput is 150 writes/s
PASS: Slowest request took 0.083526s
PASS: Stddev is 0.008879s
PASS
```

### Moving the etcd cluster leadership

MOVE-LEADER transfers leadership from the leader to another member in the cluster. This is usefull if you have to replace the etcd node which is the current leader.

Check who is the current leader

```
sh-4.2# etcdctl endpoint status --cluster -w table
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
|          ENDPOINT           |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://192.168.100.22:2379 | 570db60db6086170 |  3.3.17 |   37 MB |     false |        12 |   15776619 |
| https://192.168.100.23:2379 | ac4f6a42ebd3adfb |  3.3.17 |   37 MB |      true |        12 |   15776619 |
| https://192.168.100.21:2379 | d241e2cf814e2ec1 |  3.3.17 |   37 MB |     false |        12 |   15776619 |
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
```

The current leader is https://192.168.100.23 (IS LEADER true)
Let's move the leadership to https://192.168.100.21

We have to use for the leader the endpoint of the leader (https://192.168.100.23:2379) and for the new leader the hex ID of the member (d241e2cf814e2ec1)

```
sh-4.2# etcdctl --endpoints https://192.168.100.23:2379 move-leader d241e2cf814e2ec1
2020-04-05 14:56:06.371548 W | pkg/flags: recognized environment variable ETCDCTL_CERT, but unused: shadowed by corresponding flag
2020-04-05 14:56:06.371565 W | pkg/flags: recognized environment variable ETCDCTL_CACERT, but unused: shadowed by corresponding flag
2020-04-05 14:56:06.371570 W | pkg/flags: recognized environment variable ETCDCTL_KEY, but unused: shadowed by corresponding flag
Leadership transferred from ac4f6a42ebd3adfb to d241e2cf814e2ec1
```

Check the leadership again

```
sh-4.2# etcdctl endpoint status --cluster -w table
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
|          ENDPOINT           |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://192.168.100.22:2379 | 570db60db6086170 |  3.3.17 |   37 MB |     false |        13 |   15776832 |
| https://192.168.100.23:2379 | ac4f6a42ebd3adfb |  3.3.17 |   37 MB |     false |        13 |   15776832 |
| https://192.168.100.21:2379 | d241e2cf814e2ec1 |  3.3.17 |   37 MB |      true |        13 |   15776832 |
+-----------------------------+------------------+---------+---------+-----------+-----------+------------+
```

The current leader is now https://192.168.100.21

## etcd encryption

#### Encrypt etcd

To verify the encryption we'll set up a secret first

```
[root@bastion ~]# oc create secret generic secret1 -n default --from-literal=mykey=mydata
secret/secret1 created
```

Let's take a look how it looks like  on oc  cmdline

```
[root@bastion ~]# oc get secret secret1 -n default -o yaml
apiVersion: v1
data:
  mykey: bXlkYXRh
kind: Secret
metadata:
  creationTimestamp: "2020-04-14T15:40:37Z"
  name: secret1
  namespace: default
  resourceVersion: "6022851"
  selfLink: /api/v1/namespaces/default/secrets/secret1
  uid: dd31f75e-e0af-4974-8f31-15e6c993d11d
type: Opaque
```

Decode the value (bXlkYXRh)

```
[root@bastion ~]# echo "bXlkYXRh"|base64 -d
mydata
```

Let's take a look how it looks like on the datastore inside etcd

```
sh-4.2# etcdctl get /kubernetes.io/secrets/default/secret1| hexdump -C
00000000  2f 6b 75 62 65 72 6e 65  74 65 73 2e 69 6f 2f 73  |/kubernetes.io/s|
00000010  65 63 72 65 74 73 2f 64  65 66 61 75 6c 74 2f 73  |ecrets/default/s|
00000020  65 63 72 65 74 31 0a 6b  38 73 00 0a 0c 0a 02 76  |ecret1.k8s.....v|
00000030  31 12 06 53 65 63 72 65  74 12 67 0a 4c 0a 07 73  |1..Secret.g.L..s|
00000040  65 63 72 65 74 31 12 00  1a 07 64 65 66 61 75 6c  |ecret1....defaul|
00000050  74 22 00 2a 24 64 64 33  31 66 37 35 65 2d 65 30  |t".*$dd31f75e-e0|
00000060  61 66 2d 34 39 37 34 2d  38 66 33 31 2d 31 35 65  |af-4974-8f31-15e|
00000070  36 63 39 39 33 64 31 31  64 32 00 38 00 42 08 08  |6c993d11d2.8.B..|
00000080  f5 b2 d7 f4 05 10 00 7a  00 12 0f 0a 05 6d 79 6b  |.......z.....myk|
00000090  65 79 12 06 6d 79 64 61  74 61 1a 06 4f 70 61 71  |ey..mydata..Opaq|
000000a0  75 65 1a 00 22 00 0a                              |ue.."..|
000000a7
```

At the end you'll see the key (mykey) and the value (mydata) of the secret

Modify the API server object and set the encryption field type to aescbc

```
[root@bastion ~]# oc patch apiservers.config.openshift.io cluster --type=merge -p '{"spec": {"encryption": {"type": "aescbc"}}}'
```

The encryption wil start and it will take a while ... wait until the encryption process is completed

Review the encrypted status condition for the OpenShift API server to verify that its resources were successfully encrypted

```
[root@bastion ~]# oc get openshiftapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'
EncryptionCompleted
All resources encrypted: routes.route.openshift.io, oauthaccesstokens.oauth.openshift.io, oauthauthorizetokens.oauth.openshift.io
```

Review the Encrypted status condition for the Kubernetes API server to verify that its resources were successfully encrypted

```
[root@bastion ~]# oc get kubeapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'
EncryptionCompleted
All resources encrypted: secrets, configmaps
```

Check now the etcd entry of the secret

```
sh-4.2# etcdctl get /kubernetes.io/secrets/default/secret1| hexdump -C
00000000  2f 6b 75 62 65 72 6e 65  74 65 73 2e 69 6f 2f 73  |/kubernetes.io/s|
00000010  65 63 72 65 74 73 2f 64  65 66 61 75 6c 74 2f 73  |ecrets/default/s|
00000020  65 63 72 65 74 31 0a 6b  38 73 3a 65 6e 63 3a 61  |ecret1.k8s:enc:a|
00000030  65 73 63 62 63 3a 76 31  3a 31 3a e0 ad cb 5b f4  |escbc:v1:1:...[.|
00000040  be 1d 7e b9 84 e7 8f d7  43 b5 66 a8 7c dd 51 89  |..~.....C.f.|.Q.|
00000050  41 07 b2 6e 5e 0e 93 3e  db 0b 39 11 38 98 4c 62  |A..n^..>..9.8.Lb|
00000060  7a d0 d5 02 d2 c0 9e 70  8d 96 5d 17 d4 d4 4a f9  |z......p..]...J.|
00000070  96 52 e2 8a ec c2 90 64  f7 9c 9d 27 57 38 37 5f  |.R.....d...'W87_|
00000080  11 7e 84 7d 6d 02 8a 5f  24 35 76 44 be 32 55 96  |.~.}m.._$5vD.2U.|
00000090  31 bb ae bd b3 73 5f 5d  8e b3 2f fc 26 ab a5 a3  |1....s_]../.&...|
000000a0  84 2b a3 8a 92 d5 cb bb  e9 39 6c 4e 9d df 06 b7  |.+.......9lN....|
000000b0  b3 aa ac ec b8 76 b3 f4  40 1a 15 51 44 e7 4c d5  |.....v..@..QD.L.|
000000c0  e7 50 a3 e4 3e 84 35 ab  ee 1e cc 0a              |.P..>.5.....|
000000cc
```

We'll see that the enrty is now encrypted

Check the secret with the oc command line

```
[root@bastion ~]# oc get secret secret1 -n default -o yaml
apiVersion: v1
data:
  mykey: bXlkYXRh
kind: Secret
metadata:
  creationTimestamp: "2020-04-14T15:40:37Z"
  name: secret1
  namespace: default
  resourceVersion: "6022851"
  selfLink: /api/v1/namespaces/default/secrets/secret1
  uid: dd31f75e-e0af-4974-8f31-15e6c993d11d
type: Opaque
```

```
[root@bastion ~]# echo "bXlkYXRh"|base64 -d
mydata
```

We'll see the secret isn't encrypted, therefor only the etcd datastore is encrypted

#### Unencrypt etcd

Modify the API server object and set the encryption field type to identity:

```
[root@bastion ~]# oc patch apiservers.config.openshift.io cluster --type=merge -p '{"spec": {"encryption": {"type": "identity"}}}'
```

Review the Decrypted status condition for the OpenShift API server to verify that its resources were successfully decrypted

```
[root@bastion ~]# oc get openshiftapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'
DecryptionCompleted
Encryption mode set to identity and everything is decrypted
```

Review the Encrypted status condition for the Kubernetes API server to verify that its resources were successfully encrypted

```
[root@bastion ~]# oc get kubeapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="Encrypted")]}{.reason}{"\n"}{.message}{"\n"}'
DecryptionCompleted
Encryption mode set to identity and everything is decrypted
```

Let's take a look how it looks like on the datastore inside etcd

```
sh-4.2# etcdctl get /kubernetes.io/secrets/default/secret1| hexdump -C
00000000  2f 6b 75 62 65 72 6e 65  74 65 73 2e 69 6f 2f 73  |/kubernetes.io/s|
00000010  65 63 72 65 74 73 2f 64  65 66 61 75 6c 74 2f 73  |ecrets/default/s|
00000020  65 63 72 65 74 31 0a 6b  38 73 00 0a 0c 0a 02 76  |ecret1.k8s.....v|
00000030  31 12 06 53 65 63 72 65  74 12 67 0a 4c 0a 07 73  |1..Secret.g.L..s|
00000040  65 63 72 65 74 31 12 00  1a 07 64 65 66 61 75 6c  |ecret1....defaul|
00000050  74 22 00 2a 24 64 64 33  31 66 37 35 65 2d 65 30  |t".*$dd31f75e-e0|
00000060  61 66 2d 34 39 37 34 2d  38 66 33 31 2d 31 35 65  |af-4974-8f31-15e|
00000070  36 63 39 39 33 64 31 31  64 32 00 38 00 42 08 08  |6c993d11d2.8.B..|
00000080  f5 b2 d7 f4 05 10 00 7a  00 12 0f 0a 05 6d 79 6b  |.......z.....myk|
00000090  65 79 12 06 6d 79 64 61  74 61 1a 06 4f 70 61 71  |ey..mydata..Opaq|
000000a0  75 65 1a 00 22 00 0a                              |ue.."..|
000000a7
```

We'll see the etcd datastore is not encrypted anymore

Delete the secret secret1 to keep etcd and your OpenShift clean

```
[root@bastion ~]# oc delete secret secret1 -n default
secret "secret1" deleted
```

## etcd backup

Create a new project backup-test

```
[root@bastion ~]# oc new-project backup-test
Now using project "backup-test" on server "https://api.ocp4.h1.rhaw.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app django-psql-example

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

#### etcd backup the OpenShift way

> You should only save a backup from a single master host. You do not need a backup from each master host in the cluster.

SSH to any of the master host with the user `core`.

```
[root@bastion ~]# ssh core@master03.ocp4.h1.rhaw.io
The authenticity of host 'master03.ocp4.h1.rhaw.io (192.168.100.23)' can't be established.
ECDSA key fingerprint is SHA256:5qwHuyme0ACHc8XeG9LO9ZGVw3ePcK2M3L5F5vE6OxE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'master03.ocp4.h1.rhaw.io,192.168.100.23' (ECDSA) to the list of known hosts.
Red Hat Enterprise Linux CoreOS 43.81.202001142154.0
  Part of OpenShift 4.5, RHCOS is a Kubernetes native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).

WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.5/architecture/architecture-rhcos.html

---
```

> **`From version 4.5.6  the backup procedure has changed!`**

Run the etcd-snapshot-backup.sh script as shown below:

```
[core@master01 ~]$ sudo /usr/local/bin/etcd-snapshot-backup.sh ./assets/backup
Creating asset directory ./assets
e9aa2d19b0fcfe07d3600beab6ee275f7f67cbabdb2da1e251ff72b60e26c7e5
etcdctl version: 3.3.17
API version: 3.3
Trying to backup etcd client certs..
etcd client certs found in /etc/kubernetes/static-pod-resources/kube-apiserver-pod-9 backing up to ./assets/backup/
Backing up /etc/kubernetes/manifests/etcd-member.yaml to ./assets/backup/
Trying to backup latest static pod resources..
{"level":"warn","ts":"2020-04-17T17:09:29.264Z","caller":"clientv3/retry_interceptor.go:116","msg":"retry stream intercept"}
Snapshot saved at ./assets/backup/snapshot_2020-04-17_170928.db
snapshot db and kube resources are successfully saved to ./assets/backup!
```

The new created directory assets looks like this:

```
[root@bastion ~]# tree assets
assets
├── backup
│   ├── etcd-ca-bundle.crt
│   ├── etcd-client.crt
│   ├── etcd-client.key
│   ├── etcd-member.yaml
│   ├── snapshot_2020-04-17_170127.db
│   └── static_kuberesources_2020-04-17_170127.tar.gz
├── bin
│   └── etcdctl
├── manifests
├── restore
├── shared
├── templates
└── tmp

7 directories, 7 files
```

To preserve all permissions on the files tar it and store it on a safe place

```
[core@master01 ~]$ sudo tar cvzf backup-$(date +%Y-%m-%d).tar.gz assets/backup
assets/backup/
assets/backup/etcd-ca-bundle.crt
assets/backup/etcd-client.crt
assets/backup/etcd-client.key
assets/backup/etcd-member.yaml
assets/backup/static_kuberesources_2020-04-17_170928.tar.gz
assets/backup/snapshot_2020-04-17_170928.db
```

<!--
OLD ETCD PROCEDURE BEFORE 4.5.6 !!!
```
[core@master03 ~]$ sudo /usr/local/bin/etcd-snapshot-backup.sh ./assets/backup
Creating asset directory ./assets
Downloading etcdctl binary..
etcdctl version: 3.3.17
API version: 3.3
Trying to backup etcd client certs..
etcd client certs found in /etc/kubernetes/static-pod-resources/kube-apiserver-pod-13 backing up to ./assets/backup/
Backing up /etc/kubernetes/manifests/etcd-member.yaml to ./assets/backup/
Trying to backup latest static pod resources..
{"level":"warn","ts":"2020-04-15T09:20:33.671Z","caller":"clientv3/retry_interceptor.go:116","msg":"retry stream intercept"}
Snapshot saved at ./assets/tmp/snapshot.db
snapshot db and kube resources are successfully saved to ./assets/backup/snapshot_db_kuberesources_2020-04-15_092032.tar.gz!
```
The resulting directory assets looks like this
```
tree assets/
assets/
├── backup
│   ├── etcd-ca-bundle.crt
│   ├── etcd-member.yaml
│   └── snapshot_db_kuberesources_2020-04-15_092032.tar.gz
├── bin
│   └── etcdctl
├── manifests
├── restore
├── shared
│   ├── Documentation
│   │   ├── benchmarks
│   │   │   ├── etcd-2-1-0-alpha-benchmarks.md
│   │   │   ├── etcd-2-2-0-benchmarks.md
│   │   │   ├── etcd-2-2-0-rc-benchmarks.md
│   │   │   ├── etcd-2-2-0-rc-memory-benchmarks.md
│   │   │   ├── etcd-3-demo-benchmarks.md
│   │   │   ├── etcd-3-watch-memory-benchmark.md
│   │   │   ├── etcd-storage-memory-benchmark.md
│   │   │   └── _index.md
│   │   ├── branch_management.md
│   │   ├── demo.md
│   │   ├── dev-guide
│   │   │   ├── api_concurrency_reference_v3.md
│   │   │   ├── api_grpc_gateway.md
│   │   │   ├── api_reference_v3.md
│   │   │   ├── apispec
│   │   │   │   └── swagger
│   │   │   │       ├── rpc.swagger.json
│   │   │   │       ├── v3election.swagger.json
│   │   │   │       └── v3lock.swagger.json
│   │   │   ├── experimental_apis.md
│   │   │   ├── grpc_naming.md
│   │   │   ├── _index.md
│   │   │   ├── interacting_v3.md
│   │   │   ├── limit.md
│   │   │   └── local_cluster.md
│   │   ├── dev-internal
│   │   │   ├── discovery_protocol.md
│   │   │   ├── logging.md
│   │   │   └── release.md
│   │   ├── dl_build.md
│   │   ├── faq.md
│   │   ├── _index.md
│   │   ├── integrations.md
│   │   ├── learning
│   │   │   ├── api_guarantees.md
│   │   │   ├── api.md
│   │   │   ├── auth_design.md
│   │   │   ├── client-architecture.md
│   │   │   ├── client-feature-matrix.md
│   │   │   ├── data_model.md
│   │   │   ├── glossary.md
│   │   │   ├── _index.md
│   │   │   ├── learner.md
│   │   │   └── why.md
│   │   ├── metrics.md
│   │   ├── op-guide
│   │   │   ├── authentication.md
│   │   │   ├── clustering.md
│   │   │   ├── configuration.md
│   │   │   ├── container.md
│   │   │   ├── etcd3_alert.rules
│   │   │   ├── etcd3_alert.rules.yml
│   │   │   ├── etcd-sample-grafana.png
│   │   │   ├── failures.md
│   │   │   ├── gateway.md
│   │   │   ├── grafana.json
│   │   │   ├── grpc_proxy.md
│   │   │   ├── hardware.md
│   │   │   ├── _index.md
│   │   │   ├── maintenance.md
│   │   │   ├── monitoring.md
│   │   │   ├── performance.md
│   │   │   ├── recovery.md
│   │   │   ├── runtime-configuration.md
│   │   │   ├── runtime-reconf-design.md
│   │   │   ├── security.md
│   │   │   ├── supported-platform.md
│   │   │   ├── v2-migration.md
│   │   │   └── versioning.md
│   │   ├── platforms
│   │   │   ├── aws.md
│   │   │   ├── container-linux-systemd.md
│   │   │   ├── freebsd.md
│   │   │   └── _index.md
│   │   ├── production-users.md
│   │   ├── README.md -> docs.md
│   │   ├── reporting_bugs.md
│   │   ├── rfc
│   │   │   └── _index.md
│   │   ├── tuning.md
│   │   └── upgrades
│   │       ├── _index.md
│   │       ├── upgrade_3_0.md
│   │       ├── upgrade_3_1.md
│   │       ├── upgrade_3_2.md
│   │       ├── upgrade_3_3.md
│   │       ├── upgrade_3_5.md
│   │       └── upgrading-etcd.md
│   ├── README-etcdctl.md
│   ├── README.md
│   └── READMEv2-etcdctl.md
├── templates
└── tmp
    └── etcd-v3.3.17-linux-amd64.tar.gz
```
-->

Delete the project backup-test

```
[root@bastion ~]# oc delete project backup-test 
project.project.openshift.io "backup-test" deleted
```

#### etcd restore the backup the OpenShift way

Let's recover the etcd state before we deleted the project backup-test

First we have to copy our backup to all three masters

```

```

Than login to all three masters and run the recovery procedure simultaniously (tmux is perfect for this task)

> Hint: open a tmux session, split the window into three panes, login to one master on each pane, syncronize the panes and run the recovery procedure)

```

```