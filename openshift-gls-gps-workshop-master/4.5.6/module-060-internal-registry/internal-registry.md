## Intermediate Image Registry Storage Configuration

If the image-registry operator is not available after installation, we must configure storage for it. Instructions for both configuring a PersistentVolume, which is required for production clusters, and for configuring an empty directory as the storage location, which is available for only non-production clusters, are shown in this workshop. For now we just append dynamical storage to the registry.

First we check if we do not have a registry pod:

```sh
[root@bastion ~]# oc get pod -n openshift-image-registry
NAME READY STATUS RESTARTS AGE
cluster-image-registry-operator-56f5f56b8-ssjxj 2/2 Running 0 6m40s
```

If no image-registry pod is showing up, we need to patch the image registry operator with the following command to use local storage:

```sh
[root@bastion ~]# oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```

On platforms that do not provide shareable object storage, the OpenShift Image Registry Operator bootstraps itself as `Removed`. This allows openshift-installer to complete installations on these platform types.
After installation, you must edit the Image Registry Operator configuration to switch the ManagementState from `Removed` to `Managed`.

```sh
[root@bastion ~]# oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
config.imageregistry.operator.openshift.io/cluster patched
```

Now we can watch the creation of the image regisrty pods:

```sh
[root@bastion ~]# watch oc get pod -n openshift-image-registry
NAME                                              READY   STATUS              RESTARTS   AGE
cluster-image-registry-operator-56f5f56b8-ssjxj   2/2     Running             0          8m34s
image-registry-57944b948b-42jvh                   0/1     ContainerCreating   0          6s
image-registry-64d649744c-bhn7k                   0/1     ContainerCreating   0          6s
node-ca-gn8v8                                     0/1     ContainerCreating   0          6s
node-ca-mzbwz                                     0/1     ContainerCreating   0          6s
node-ca-pxnwx                                     0/1     ContainerCreating   0          6s
node-ca-ql7s5                                     0/1     ContainerCreating   0          7s
node-ca-wql85                                     0/1     ContainerCreating   0          6s
```

The pods should now be up and running.

## Container Image Registry

The registry uses a similar CRD (Custom Resource Definition) mechanism to
configure how the operator deploys the actual registry pods. That CRD is
configs.imageregistry.operator.openshift.io. You will need to edit the cluster
CR object in order to add the nodeSelector.

- First, take a look at it:

```yaml
oc get configs.imageregistry.operator.openshift.io/cluster -o yaml
apiVersion: imageregistry.operator.openshift.io/v1
kind: Config
metadata:
  creationTimestamp: "2020-02-11T16:06:29Z"
  finalizers:
  - imageregistry.operator.openshift.io/finalizer
  generation: 3
  name: cluster
  resourceVersion: "22151"
  selfLink: /apis/imageregistry.operator.openshift.io/v1/configs/cluster
  uid: 3d602020-fb6d-4974-9643-9b381672c68d
spec:
  defaultRoute: false
  disableRedirect: false
  httpSecret: 0428bc17f51665ba5f7b9d73d15fac7043783fb5918f8bc5a7f3323f24eb6488a63e69178758357bc2da106430df2c89bebd2d667893a5cf0a192a5d717e3ad9
  logging: 2
  managementState: Managed
  proxy:
    http: ""
    https: ""
    noProxy: ""
  readOnly: false
  replicas: 1
  requests:
    read:
      maxInQueue: 0
      maxRunning: 0
      maxWaitInQueue: 0s
    write:
      maxInQueue: 0
      maxRunning: 0
      maxWaitInQueue: 0s
  storage:
    emptyDir: {}
status:
  conditions:
  - lastTransitionTime: "2020-02-11T16:28:34Z"
    message: The registry is ready
    reason: Ready
    status: "False"
    type: Progressing
  - lastTransitionTime: "2020-02-11T16:28:34Z"
    message: The registry is ready
    reason: Ready
    status: "True"
    type: Available
  - lastTransitionTime: "2020-02-11T16:06:29Z"
    status: "False"
    type: Degraded
  - lastTransitionTime: "2020-02-11T16:28:02Z"
    status: "False"
    type: Removed
  - lastTransitionTime: "2020-02-11T16:28:00Z"
    message: EmptyDir storage successfully created
    reason: Creation Successful
    status: "True"
    type: StorageExists
  observedGeneration: 3
  readyReplicas: 0
  storage:
    emptyDir: {}
  storageManaged: false
```

Next, let's modify the custom resource by live-patching the configuration.
For this we can use oc edit, and you'll need to modify the .spec section:

----

```sh
oc edit configs.imageregistry.operator.openshift.io/cluster
```

The .spec section will need to look like the following:

```yaml
...
  nodeSelector:
    node-role.kubernetes.io/infra: ""
...
```

----

- Once you're done, save and exit the editor, and it should confirm the change:

```sh
config.imageregistry.operator.openshift.io/cluster edited
```

> NOTE: The nodeSelector stanza may be added anywhere inside the .spec block.

Same can be achieved by running the following oc patch command

```sh
oc patch configs.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"nodeSelector":{"node-role.kubernetes.io/infra": ""}}}'
```

When you save and exit you should see the registry pod being moved to the infra
node. The registry is in the openshift-image-registry project. If you execute
the following quickly enough, you may see the old registry pods terminating and
the new ones starting.:

```sh
$ oc get pod -n openshift-image-registry
NAME                                               READY   STATUS        RESTARTS   AGE
cluster-image-registry-operator-5644775d7c-w78kh   1/1     Running       0          34h
image-registry-5878c9d896-nmkc6                    1/1     Terminating   0          22h
node-ca-2ljck                                      1/1     Running       0          22h
node-ca-9npbz                                      1/1     Running       0          34h
node-ca-mk9lj                                      1/1     Running       0          34h
node-ca-pspwx                                      1/1     Running       0          34h
node-ca-qlxqx                                      1/1     Running       0          9h
node-ca-qvslw                                      1/1     Running       0          34h
node-ca-wxb55                                      1/1     Running       0          34h
node-ca-xn9vg                                      1/1     Running       0          22h
```

> NOTE: At this time the image registry is not using a separate project for its operator. Both the operator and the operand are housed in the openshift-image-registry project.

Since the registry is being backed by an S3 bucket, it doesn't matter what node the new registry pod instance lands on. It's talking to an object store via an API, so any existing images stored there will remain accessible.

Also note that the default replica count is 1. In a real-world environment you might wish to scale that up for better availability, network throughput, or other reasons.

If you look at the node on which the registry landed (noting that you'll likely have to refresh your list of pods by using the previous commands to get its new name):

```sh
$ oc get pod image-registry-58657f5d4d-hmjph -n openshift-image-registry -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
image-registry-58657f5d4d-hmjph   1/1     Running   0          2d21h   10.128.2.15   node02   <none>           <none>
```

> NOTE: the pod name will be different in your environment

it is now running on an infra node:

```sh
$ oc get node node02
NAME       STATUS   ROLES    AGE     VERSION
node02   Ready    infra    2d21h   v1.16.2
```

> Notice that the CRD for the image registry's configuration is not
> namespaced -- it is cluster scoped. There is only one internal/integrated
> registry per OpenShift cluster that serves all projects.

## 
