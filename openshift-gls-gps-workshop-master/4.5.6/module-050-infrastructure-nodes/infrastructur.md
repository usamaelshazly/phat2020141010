# Postinstallation Infrastructure Nodes

In this Postinstallation Task, we will turn node Nodes into Infrastructure Nodes.

The reason is that after the initial deployment,  an OpenShift 4 Cluster just include master and node nodes, while OpenShift 3 used to ship with master, infra and compute nodes.

The node nodes in OpenShift 4 are meant to replace both infra and node nodes, which could make sense running smaller setups, though we would argue that is not practical to scale out in larger environments with hundrets of node nodes. Having a small set of nodes, designated to host OpenShift ingress controllers is a good thing, as we only need to configure those IPs as backends for our applications loadbalancers. If we have not seperated infra from  node nodes, every time we add new members to our cluster, we would also need reconfiguring our loadbalancer. At the end we could have hundrets of possible node nodes on our external Loadbalancer acts as a possible endpoint for Openshift Routers.

To prevent this we would create a group of Infra machines, starting with creating a MachineConfigPool, using the following configuration

```sh
[root@bastion ~]# vim /root/openshift/machineconfiguration/infra-machineconfigpool.yaml
```

```yaml
---
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: infra
spec:
  machineConfigSelector:
    matchExpressions:
    - key: machineconfiguration.openshift.io/role
      operator: In
      values:
      - node
      - infra
  maxUnavailable: 1
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/infra: ""
  paused: false
...
```

We created a machineconfigpool infra that inherits everything from also in the machineconfigpool node.

After crerating the configuration file we need to apply this to our cluster:

```sh
[root@bastion ~]# oc create -f infra-machineconfigpool.yaml
```

```
oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT
infra                                                                                       0              0                   0                     0
master   rendered-master-9b499c1cfc683a5ba9a54c47ce7570d3   True      False      False      3              3                   3                     0
node   rendered-node-da58576c03c205503967c5fdf465ab80   True      False      False      4              4                   4                     0
```

As you can see in the output the mcp infra has a machinecount of 0.

To add machines to the newly created pool we have to label the machies we want to move to the pool:

```sh
[root@bastion ~]# oc label node node03 node-role.kubernetes.io/infra=
node/node03 labeled
```

and:

```sh
[root@bastion ~]# oc label node node03 node-role.kubernetes.io/node-
node/node03 labeled
```

From there, our node would be set unschedulable, drained, and rebooted.

```sh
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    master,node   30h   v1.16.2
master02   Ready    master,node   30h   v1.16.2
master03   Ready    master,node   30h   v1.16.2
node01   Ready    node          30h   v1.16.2
node02   Ready    node          30h   v1.16.2
node03   Ready    infra           13h   v1.16.2
node04   Ready    infra           13h   v1.16.2
```

When our nodes is back, we would proceed with the next step.
The oc wait command is here a nice help:

```
oc wait mcp/infra --for condition=updated --timeout=-1s
```

Check after the machineconfigpool infra is updated it looks like this:

```
[root@bastion ~]# oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT
infra    rendered-infra-52ec490315c0f8fb1d384a363731c05f    True      False      False      2              2                   2                     0
master   rendered-master-9b499c1cfc683a5ba9a54c47ce7570d3   True      False      False      3              3                   3                     0
node   rendered-node-da58576c03c205503967c5fdf465ab80   True      False      False      2              2                   2                     0
```

We would eventually reconfigure our Ingress Controller deploying OpenShift Routers back to our infra nodes:

```sh
[root@service ~]# oc edit -n openshift-ingress-operator ingresscontroller default
```

```yaml
...
spec:
 nodePlacement:
   nodeSelector:
     matchLabels:
       node-role.kubernetes.io/infra: ""
 replicas: 2
...
```

or run an oc patch command

```sh
[root@bastion ~]# oc patch ingresscontrollers.operator.openshift.io default -n openshift-ingress-operator -p '{"spec":{"nodePlacement":{"nodeSelector":{"matchLabels":{"node-role.kubernetes.io/infra":""}}}}}' --type=merge
ingresscontroller.operator.openshift.io/default patched
```

We would then keep track of routers pods as they’re being re-deployed to the infra nodes

```sh
[root@service ~]# oc get pods -n openshift-ingress -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
router-default-7d66988b48-rkwfm   1/1     Running   0          30h   192.168.100.33   node03   <none>           <none>
router-default-7d66988b48-xsl2k   1/1     Running   0          30h   192.168.100.34   node04   <none>           <none>
```

By default there are two router, if you have to have more e.g. three infra nodes modify once again the Ingress Controller

```sh
[root@bastion ~]# oc patch ingresscontrollers.operator.openshift.io default -n openshift-ingress-operator --patch '{"spec":{"replicas": 3}}' --type=merge
ingresscontroller.operator.openshift.io/default patched
```

> Do not forget to reconfigure the loadbalancer so it will point to the infras!

Check the amount of routers

```sh
[root@bastion ~]# oc get pods -n openshift-ingress -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
router-default-7d66988b48-9fg6b   1/1     Running   0          20s   192.168.100.33   node03   <none>           <none>
router-default-7d66988b48-k485m   1/1     Running   0          11m   192.168.100.34   node04   <none>           <none>
router-default-7d66988b48-vvss7   1/1     Running   0          11m   192.168.100.35   node05   <none>           <none>
```

Place the image registry pod on the infra node

```sh
[root@bastion ~]# oc patch configs.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"nodeSelector":{"node-role.kubernetes.io/infra": ""}}}'
```

Check the image registry pod placement

```sh
[root@bastion ~]# oc get pods -n openshift-image-registry -o wide
NAME                                              READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
cluster-image-registry-operator-f9697f69d-fzq5j   2/2     Running   0          30h   10.129.0.14   master03   <none>           <none>
image-registry-6f5d69f654-jxh6h                   1/1     Running   0          58s   10.128.2.18   node39   <none>           <none>
node-ca-52ts7                                     1/1     Running   0          14h   10.131.0.41   master01   <none>           <none>
node-ca-8w2pw                                     1/1     Running   1          14h   10.130.2.3    node04   <none>           <none>
node-ca-l8qhg                                     1/1     Running   0          14h   10.128.2.17   node02   <none>           <none>
node-ca-lcbk5                                     1/1     Running   0          14h   10.128.0.16   node01   <none>           <none>
node-ca-nwt2x                                     1/1     Running   1          14h   10.129.2.3    node03   <none>           <none>
node-ca-qcbsg                                     1/1     Running   0          14h   10.129.0.30   master03   <none>           <none>
node-ca-vm94k                                     1/1     Running   0          14h   10.130.0.42   master02   <none>           <none>
```

Now we are done, we have seperated node from infra nodes in our Openshift Cluster.

If you want to have different configurations on infra nodes and node nodes e.g. on the network device settings you have to define also a new "node" pool, that's call it "compute".

```sh
[root@bastion ~]# vim /root/openshift/machineconfiguration/compute-machineconfigpool.yaml
```

```yaml
---
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: compute
spec:
  machineConfigSelector:
    matchExpressions:
    - key: machineconfiguration.openshift.io/role
      operator: In
      values:
      - node
      - compute
  maxUnavailable: 1
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/compute: ""
  paused: false
...
```

We created a machineconfigpool compute that inherits everything from the machineconfigpool node.

After crerating the configuration file we need to apply this to our cluster:

```sh
[root@bastion ~]# oc create -f compute-machineconfigpool.yaml
```

```
oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT
infra      rendered-infra-52ec490315c0f8fb1d384a363731c05f  True      False      False      2              2                   2                     0
compute                                                                                     0              0                   0                     0
master   rendered-master-9b499c1cfc683a5ba9a54c47ce7570d3   True      False      False      3              3                   3                     0
node   rendered-node-da58576c03c205503967c5fdf465ab80   True      False      False      2              2                   2                     0
```

As you can see in the output the mcp compute has a machinecount of 0.

To add machines to the newly created pool we have to label the machies we want to move to the pool:

```sh
[root@bastion ~]# oc label node node01 node-role.kubernetes.io/compute=
node/node01 labeled
```

and:

```sh
[root@bastion ~]# oc label node node01 node-role.kubernetes.io/node-
node/node03 labeled
```

From there, our node would be set unschedulable, drained, and rebooted.

```sh
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES     AGE   VERSION
master01   Ready    master    30h   v1.16.2
master02   Ready    master    30h   v1.16.2
master03   Ready    master    30h   v1.16.2
node01   Ready    compute   30h   v1.16.2
node02   Ready    compute   30h   v1.16.2
node03   Ready    infra     13h   v1.16.2
node04   Ready    infra     13h   v1.16.2
```

When our nodes is back, we would proceed with the next step.
The oc wait command is here a nice help:

```
oc wait mcp/infra --for condition=updated --timeout=-1s
```

Check after the machineconfigpool compute is updated it looks like this:

```
[root@bastion ~]# oc get mcp
NAME       CONFIG                                               UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT
infra      rendered-infra-52ec490315c0f8fb1d384a363731c05f      True      False      False      2              2                   2                     0
compute    rendered-compute-52ec490315c0f8fb1d384a363731c05f    True      False      False      2              2                   2                     0
master     rendered-master-9b499c1cfc683a5ba9a54c47ce7570d3     True      False      False      3              3                   3                     0
node     rendered-node-da58576c03c205503967c5fdf465ab80     True      False      False      0              0                   0                     0
```
