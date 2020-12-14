After we have done the installation we can check the cluster is up and running type in the following command:

```sh
[root@bastion ~]# export KUBECONFIG=/root/ocp4/auth/kubeconfig
```

```sh
[root@bastion ~]# oc whoami
system:admin
```

```sh
[root@bastion ~]# oc get nodes
NAME       STATUS   ROLES           AGE   VERSION
master01   Ready    master,node   37h   v1.16.2
master02   Ready    master,node   37h   v1.16.2
master03   Ready    master,node   37h   v1.16.2
node01   Ready    node          37h   v1.16.2
node02   Ready    node          37h   v1.16.2
node03   Ready    node          21h   v1.16.2
```

You should get an output of six machines in state Ready.

## Troubleshooting: Pending  Certificates

When you add machines to a cluster, two pending certificates signing request (CSRs) are generated for each machine that you added. You must verify that these CSRs are approved or, if necessary, approve them yourself.

```sh
[root@bastion ~]# oc get nodes
NAME       STATUS      ROLES           AGE   VERSION
master01   Ready       master,node   37h   v1.16.2
master02   Ready       master,node   37h   v1.16.2
master03   Ready       master,node   37h   v1.16.2
node01   Ready       node          37h   v1.16.2
node02   NotReady    node          37h   v1.16.2
node03   NotReady    node          21h   v1.16.2
...
```

The output lists all of the machines that we created.

Now we need to review the pending certificate signing requests (CSRs) and ensure that the you see a client and server request with `Pending` or `Approved` status for each machine that you added to the cluster:

```sh
[root@bastion ~]# oc get csr
NAME        AGE     REQUESTOR                                                                   CONDITION
csr-8b2br   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-8vnps   15m     system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Pending
csr-bfd72   5m26s   system:node:ip-10-0-50-126.us-east-2.compute.internal                       Pending
csr-c57lv   5m26s   system:node:ip-10-0-95-157.us-east-2.compute.internal                       Pending
```

> |
> Because the CSRs rotate automatically, approve your CSRs within an hour of adding the machines to the cluster. If you do not approve them within an hour, the certificates will rotate, and more than two certificates will be present for each node. You must approve all of these certificates. After you approve the initial CSRs, the subsequent node client CSRs are automatically approved by the cluster kube-controller-manager. You must implement a method of automatically approving the kubelet serving certificate requests.

Now we need to approve pending certificates:

```sh
[root@bastion ~]# oc adm certificate approve csr-bfd72
```

Tip:
To approve all pending certificates run the folloing command:

```sh
[root@bastion ~]# oc get csr -o name | xargs oc adm certificate approve
```

After that we can check the csr status again and validate that they are all "Approved,Issued":

```sh
[root@bastion ~]# oc get csr
```

## Completing installation on User Provisioned Infrastructure:

After we complete the operator configuration, you can finish installing the cluster on infrastructure that you provide.

We need to confirm that all components are up and running.

```sh
[root@bastion ~]# watch -n5 oc get clusteroperators
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.5.6     True        False         False      10m
cloud-credential                           4.5.6     True        False         False      22m
cluster-autoscaler                         4.5.6     True        False         False      21m
console                                    4.5.6     True        False         False      10m
dns                                        4.5.6     True        False         False      21m
image-registry                             4.5.6     True        False         False      16m
ingress                                    4.5.6     True        False         False      16m
insights                                   4.5.6     True        False         False      19m
kube-apiserver                             4.5.6     True        False         False      18m
kube-controller-manager                    4.5.6     True        False         False      22m
kube-scheduler                             4.5.6     True        False         False      22m
machine-api                                4.5.6     True        False         False      18m
machine-config                             4.5.6     True        False         False      18m
marketplace                                4.5.6     True        False         False      18m
monitoring                                 4.5.6     True        False         False      16m
network                                    4.5.6     True        False         False      21m
node-tuning                                4.5.6     True        False         False      21m
openshift-apiserver                        4.5.6     True        False         False      17m
openshift-controller-manager               4.5.6     True        False         False      14m
openshift-samples                          4.5.6     True        False         False      21m
operator-lifecycle-manager                 4.5.6     True        False         False      21m
operator-lifecycle-manager-catalog         4.5.6     True        False         False      21m
operator-lifecycle-manager-packageserver   4.5.6     True        False         False      21m
service-ca                                 4.5.6     True        False         False      16m
service-catalog-apiserver                  4.5.6     True        False         False      16m
service-catalog-controller-manager         4.5.6     True        False         False      16m
storage                                    4.5.6     True        False         False      17m
```

  When all of the cluster Operators are available (the kube-apiserver operator is last in state PROGRESSING=True and takes roughly 15min to finish), we can complete the installation.

> The Ignition config files that the installation program generates contain certificates that expire after 24 hours. You must keep the cluster running for 24 hours in a non-degraded state to ensure that the first certificate rotation has finished

## Preparing your cluster to gather support data

Clusters using a restricted network must import the default must-gather image in order to gather debugging data for Red Hat support. The must-gather image is not imported by default, and clusters on a restricted network do not have access to the internet to pull the latest image from a remote repository.

Import the default must-gather image from your installation payload:

```
[root@bastion ~]# oc import-image is/must-gather -n openshift
```

## Using Samples Operator imagestreams with alternate or mirrored registries

Most imagestreams in the OpenShift namespace managed by the Samples Operator point to images located in the Red Hat registry at [registry.redhat.io](https://registry.redhat.io/). Mirroring will not apply to these imagestreams.

> The `jenkins`, `jenkins-agent-maven`, and `jenkins-agent-nodejs` imagestreams come from the install payload and are managed by the Samples Operator, so no further mirroring procedures are needed for those imagestreams.  
> 
> Setting the `samplesRegistry` field in the Sample Operator configuration file to [registry.redhat.io](https://registry.redhat.io/) is redundant because it is already directed to [registry.redhat.io](https://registry.redhat.io/) for everything but Jenkins images and imagestreams. It also breaks the installation payload for Jenkins imagestreams.  
> 
> The Samples Operator prevents the use of the following registries for the Jenkins imagestreams:  
> 
> - [docker.io](https://docker.io/)  
> - [registry.redhat.io](https://registry.redhat.io/)  
> - [registry.access.redhat.com](https://registry.access.redhat.com/)  
> - [quay.io](https://quay.io/).
> 
> The `cli`, `installer`, `must-gather`, and `tests` imagestreams, while part of the install payload, are not managed by the Samples Operator. These are not addressed in this procedure.

### Prerequisites:

- Access to the cluster as a user with the `cluster-admin` or kubeadmin role.

- Create a pull secret for your mirror registry.

Access the images of a specific imagestream to mirror, for example:

```
[root@bastion ~]# oc get is <imagestream> -n openshift -o json | jq .spec.tags[].from.name | grep registry.redhat.io
```

Mirror images from [registry.redhat.io](https://registry.redhat.io/) associated with any imagestreams you need in the restricted network environment into one of the defined mirrors, for example:

```
[root@bastion ~]# oc image mirror registry.redhat.io/rhscl/ruby-25-rhel7:latest ${MIRROR_ADDR}/rhscl/ruby-25-rhel7:latest
```

Update the `samplesRegistry` field in the Samples Operator configuration object to contain the `hostname` portion of the mirror location defined in the mirror configuration:

> This is required because the imagestream import process does not use the mirror or search mechanism at this time.

Add any imagestreams that are not mirrored into the `skippedImagestreams` field of the Samples Operator configuration object. Or if you do not want to support any of the sample imagestreams, set the Samples Operator to `Removed` in the Samples Operator configuration object.

> Any unmirrored imagestreams that are not skipped, or if the Samples Operator is not changed to `Removed`, will result in the Samples Operator reporting a `Degraded` status two hours after the imagestream imports start failing.

Many of the templates in the OpenShift namespace reference the imagestreams. So using `Removed` to purge both the imagestreams and templates will eliminate the possibility of attempts to use them if they are not functional because of any missing imagestreams.
