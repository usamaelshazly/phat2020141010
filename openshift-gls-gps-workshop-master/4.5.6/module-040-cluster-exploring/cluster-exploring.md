## Exploring the Cluster

In this module, we will review the OpenShift Web Console.

Let's take some time to explore our OCP4 cluster. We now have two options, we can use our local terminal on our laptop, or we can use the browser-based terminal that we provisioned in the previous section. Due to connectivity challenges, we maybe forced to use the browser-based one, and for convenience our recommendation would be to use it. If we really want to configure our local client we can do so by using the following instructions to download the command line tooling. We should only do this if we don't want to use the browser-based terminal; we need to make sure we run this on our local laptop and NOT within the web-browser.

Currently using the OC client CLI from https://access.redhat.om. We can
check if there is an updated one before running the download instruction.We can
update the URL accordingly if you like.

#### For Linux:

```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.5.6/openshift-client-linux-4.5.6.tar.gz
...
oc version
```

#### For Mac  OSX:

```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.5.6/openshift-client-mac-4.5.6.tar.gz
...

oc version
```

#### For Windows:

- Download CLI from https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.5.6/openshift-client-windows-4.5.6.zip
- Extract the downloaded zip file.
- Download download git bash for Windows from http://git-scm.com is recommended
- Set up PATH on your Windows laptop

## Login via CLI

Let's now configure our command line tooling to point to our new cluster.
Below you'll need to enter the API URI, which will be shown as the "Openshift API for command line 'oc' client".
OCP4 CLI login require the [https://api.ocp4.hX.rhaw.io:6443](https://api.ocp4.hX.rhaw.io:6443) not the master URL.

```
$ oc login --server https://api.ocp4.hX.rhaw.io:6443
```

```
...
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y

Authentication required for https://api.ocp4.hX.rhaw.io:6443 (openshift)
Username: <your username>
Password: <your password>
Login successful.
...
```

We can now check that your config has been written successfully:

```
$ cat ~/.kube/config
```

```
apiVersion: v1
clusters:
cluster:
 insecure-skip-tls-verify: true
 server: https://api.ocp4.hX.rhaw.io:6443
```

Now that your client CLI is installed, you will have access to the web console and can use the CLI. Below are some command-line exercises to explore the cluster.

### Cluster Nodes

The default installation behavior creates 6 nodes: 3 masters and 3 "node" application/compute nodes. You can view them with:

```
$ oc get nodes -o wide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                                                       KERNEL-VERSION                CONTAINER-RUNTIME
master01   Ready    master,node   2d2h   v1.16.2   192.168.100.21   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
master02   Ready    master,node   2d2h   v1.16.2   192.168.100.22   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
master03   Ready    master,node   2d2h   v1.16.2   192.168.100.23   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
node01   Ready    node          2d2h   v1.16.2   192.168.100.31   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
node02   Ready    node          2d2h   v1.16.2   192.168.100.32   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
node03   Ready    node          25h    v1.16.2   192.168.100.33   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
node04   Ready    node          25h    v1.16.2   192.168.100.34   <none>        Red Hat Enterprise Linux CoreOS 43.81.202001142154.0 (Ootpa)   4.18.0-147.3.1.el8_1.x86_64   cri-o://1.16.2-6.dev.rhaos4.5.git9e3db66.el8
```

If you want to see the various applied labels, you can also do:

```
$ oc get nodes --show-labels
NAME       STATUS   ROLES           AGE    VERSION   LABELS
master01   Ready    master,node   2d2h   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master01,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
master02   Ready    master,node   2d2h   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master02,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
master03   Ready    master,node   2d2h   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master03,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
node01   Ready    node          2d2h   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
node02   Ready    node          2d2h   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node02,kubernetes.io/os=linux,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
node03   Ready    node          25h    v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node03,kubernetes.io/os=linux,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
node04   Ready    node          25h    v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node04,kubernetes.io/os=linux,node-role.kubernetes.io/node=,node.openshift.io/os_id=rhcos
```

For reference, labels are used as a mechanism to tag certain information onto a node, or a set of nodes, that can help you identify your systems, e.g. by operating system, system architecture, specification, location of the system (e.g. region), it's hostname, etc. They can also help with application scheduling, e.g. make sure that my application (or pod) resides on a specific system type. The labels shown above are utilising the default labels, but it's possible to set some custom labels in the form of a key-value pair.

### The Cluster Operator

The cluster version operator is the core of what defines an OpenShift deployment . The cluster version operator pod(s) contains the set of manifests which are used to deploy, updated, and/or manage the OpenShift bastion in the cluster. This operator ensures that the other bastion, also deployed as operators, are at the version which matches the release definition and takes action to remedy discrepancies when necessary.

```
$ oc get deployments -n openshift-cluster-version
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
cluster-version-operator   1/1     1            1           2d2h
```

You can also view the current version of the OpenShift cluster and give you a high-level indication of the status:

```
$ oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.5.6     True        False         2d1h    Cluster version is 4.5.6
```

If you want to review a list of operators that the cluster version operator is controlling, along with their status, you can ask for a list of the cluster operators:

```
$ oc get clusteroperator
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.5.6     True        False         False      2d1h
cloud-credential                           4.5.6     True        False         False      2d2h
cluster-autoscaler                         4.5.6     True        False         False      2d1h
console                                    4.5.6     True        False         False      2d1h
dns                                        4.5.6     True        False         False      2d2h
image-registry                             4.5.6     True        False         False      2d1h
ingress                                    4.5.6     True        False         False      2d1h
insights                                   4.5.6     True        False         False      2d2h
kube-apiserver                             4.5.6     True        False         False      2d1h
kube-controller-manager                    4.5.6     True        False         False      2d1h
kube-scheduler                             4.5.6     True        False         False      2d2h
machine-api                                4.5.6     True        False         False      2d2h
machine-config                             4.5.6     True        False         False      2d2h
marketplace                                4.5.6     True        False         False      2d1h
monitoring                                 4.5.6     True        False         False      31h
network                                    4.5.6     True        False         False      2d2h
node-tuning                                4.5.6     True        False         False      2d1h
openshift-apiserver                        4.5.6     True        False         False      2d1h
openshift-controller-manager               4.5.6     True        False         False      2d1h
openshift-samples                          4.5.6     True        False         False      2d1h
operator-lifecycle-manager                 4.5.6     True        False         False      2d2h
operator-lifecycle-manager-catalog         4.5.6     True        False         False      2d2h
operator-lifecycle-manager-packageserver   4.5.6     True        False         False      31h
service-ca                                 4.5.6     True        False         False      2d2h
service-catalog-apiserver                  4.5.6     True        False         False      2d1h
service-catalog-controller-manager         4.5.6     True        False         False      2d1h
storage                                    4.5.6     True        False         False      2d1h
```

Or a more comprehensive way of getting a list of operators running on the cluster, along with the link to the code, the documentation, and the commit that provided the functionality is as follows:

```
$ oc adm release info --commits
Name:      4.5.6
Digest:    sha256:3a516480dfd68e0f87f702b4d7bdd6f6a0acfdac5cd2e9767b838ceede34d70d
Created:   2020-01-22T02:05:15Z
OS/Arch:   linux/amd64
Manifests: 364

Pull From: quay.io/openshift-release-dev/ocp-release@sha256:3a516480dfd68e0f87f702b4d7bdd6f6a0acfdac5cd2e9767b838ceede34d70d

Release Metadata:
  Version:  4.5.6
  Upgrades: 4.2.16, 4.5.6-rc.0, 4.5.6-rc.1, 4.5.6-rc.2, 4.5.6-rc.3
  Metadata:
    description: 
  Metadata:
    url: https://access.redhat.com/errata/RHBA-2020:0062

Component Versions:
  Kubernetes 1.16.2

Images:
  NAME                                          REPO                                                                       COMMIT 
  aws-machine-controllers                       https://github.com/openshift/cluster-api-provider-aws                      b59f34398847fd2047f4f1193654b505643afe79
  azure-machine-controllers                     https://github.com/openshift/cluster-api-provider-azure                    86ecb23daa83c78761414a7abcb169c73c18a756
  baremetal-installer                           https://github.com/openshift/installer                                     2055609f95b19322ee6cfdd0bea73399297c4a3e
  baremetal-machine-controllers                 https://github.com/openshift/cluster-api-provider-baremetal                a2a477909c1d518ef7cf28601e5d7db56a4d4069
  baremetal-operator                            https://github.com/openshift/baremetal-operator                            d981ea02fc551aeda75ccbf11da34ba9eed11f77
  baremetal-runtimecfg                          https://github.com/openshift/baremetal-runtimecfg                          4a9707af8d6e018b652928c8141d25b5f1744123
  cli                                           https://github.com/openshift/oc                                            6a937dfe56ff26255d09702c69b8406040c14505
  cli-artifacts                                 https://github.com/openshift/oc                                            6a937dfe56ff26255d09702c69b8406040c14505
  cloud-credential-operator                     https://github.com/openshift/cloud-credential-operator                     835d9b25e076b4f058bf638a6fe61d622086d78d
  cluster-authentication-operator               https://github.com/openshift/cluster-authentication-operator               df8cd95ed3249e0fae70ddaea56c07950b819cad
  cluster-autoscaler                            https://github.com/openshift/kubernetes-autoscaler                         d45f94da59de0f949b8d84e67f7c4074ed6e0952
  cluster-autoscaler-operator                   https://github.com/openshift/cluster-autoscaler-operator                   3752471e05fb4a7ed923bef281109eae82eb9d75
  cluster-bootstrap                             https://github.com/openshift/cluster-bootstrap                             106f76b4d183cc24e58d26b3b57b2bf1e2362a9d
  cluster-config-operator                       https://github.com/openshift/cluster-config-operator                       d5ddbcdd4ec87bd4d0d109e81ed0be5dafd86061
  cluster-dns-operator                          https://github.com/openshift/cluster-dns-operator                          6f4a6005c8fbb56586b542d179b5ca1c96345498
  cluster-image-registry-operator               https://github.com/openshift/cluster-image-registry-operator               1967d568e1fa6f2919ce964b69d441f18e64b8b4
  cluster-ingress-operator                      https://github.com/openshift/cluster-ingress-operator                      db62fb9e2656ba92059beeef3241b191e062d9a2
  cluster-kube-apiserver-operator               https://github.com/openshift/cluster-kube-apiserver-operator               62da7e6b5d19114d32f5cf905a590edb9f4def0b
  cluster-kube-controller-manager-operator      https://github.com/openshift/cluster-kube-controller-manager-operator      7c430e3e821ec90916a205b71f5dfc6daf919ce2
  cluster-kube-scheduler-operator               https://github.com/openshift/cluster-kube-scheduler-operator               da32b6a109a7729678c889935ff310809263077a
  cluster-machine-approver                      https://github.com/openshift/cluster-machine-approver                      86bdd875cd12c8598a5a7790bc1e7c6e348476c4
  cluster-monitoring-operator                   https://github.com/openshift/cluster-monitoring-operator                   716a1868caa4c5800e35500f53667b12af9d66d9
  cluster-network-operator                      https://github.com/openshift/cluster-network-operator                      5b7f073de0f3ced9e9b83bbaa7cab0ade8b7d01d
  cluster-node-tuned                            https://github.com/openshift/openshift-tuned                               4de824748a0fa2dbf4b10f82a32e3784372bce54
  cluster-node-tuning-operator                  https://github.com/openshift/cluster-node-tuning-operator                  769ba5c3cf596ccb0b17e0e47b0d413850798b5c
  cluster-openshift-apiserver-operator          https://github.com/openshift/cluster-openshift-apiserver-operator          9416571f9de98516e9bb13e3d9276259adf99aff
  cluster-openshift-controller-manager-operator https://github.com/openshift/cluster-openshift-controller-manager-operator 10fc162e4f4c977a9c431884b9a063fa751514d0
  cluster-policy-controller                     https://github.com/openshift/cluster-policy-controller                     9b6ccc80eeef2eb85f78f8a70f319cbae0c5295a
  cluster-samples-operator                      https://github.com/openshift/cluster-samples-operator                      8ceee7276335336d625594f399b2ba116d6fd9d4
  cluster-storage-operator                      https://github.com/openshift/cluster-storage-operator                      a9548e6d0ac621a5c6e8e3544ca680280a1ca746
  cluster-svcat-apiserver-operator              https://github.com/openshift/cluster-svcat-apiserver-operator              61c2f1d39a8c876aa5fd7ebc14ffa9b4e1360a5f
  cluster-svcat-controller-manager-operator     https://github.com/openshift/cluster-svcat-controller-manager-operator     727d37d929cb388744362c98fbcaf9ef547cd4b6
  cluster-update-keys                           https://github.com/openshift/cluster-update-keys                           cca4ce696383e70ae669e770bd63265a9540b721
  cluster-version-operator                      https://github.com/openshift/cluster-version-operator                      beee410fc8780e5613c09fc2690716b711747041
  configmap-reloader                            https://github.com/openshift/configmap-reload                              841b0bfe743999fcc9b0528ea689f728eb92aee7
  console                                       https://github.com/openshift/console                                       a7b51a5573cf36a0d0a88aa30ee0deb8e7c0da10
  console-operator                              https://github.com/openshift/console-operator                              cb47e810bee8c56f0738f0723c98cf3192ea7736
  container-networking-plugins                  https://github.com/openshift/containernetworking-plugins                   386465fe170c0e7299b357efbace235a3e5ce7ee
  coredns                                       https://github.com/openshift/coredns                                       51569b24b4a92d6d8df2020c2e8e2866c5e78266
  deployer                                      https://github.com/openshift/oc                                            6a937dfe56ff26255d09702c69b8406040c14505
  docker-builder                                https://github.com/openshift/builder                                       50c6e17276dfc7d0acd9d943dac340fb4fa3ff24
  docker-registry                               https://github.com/openshift/image-registry                                8b12e0a3023dd39e59657984ba50c92a74f94dd5
  etcd                                          https://github.com/openshift/etcd                                          e30a18f29c8bf1f16507d7983dde2aa51962d6c7
  gcp-machine-controllers                       https://github.com/openshift/cluster-api-provider-gcp                      537d68ec0751a812755779f31d28155cc2afc454
  grafana                                       https://github.com/openshift/grafana                                       e2261e142909cb0fe70fc0996d7d375e57956589
  haproxy-router                                https://github.com/openshift/router                                        1526917a58021c8f6e465406501c32fa3fb618b5
  hyperkube                                     https://github.com/openshift/ose                                           0aee6a89a377364a4c73ab1c67bfb4bb35600a59
  insights-operator                             https://github.com/openshift/insights-operator                             6b7509911b5cf9bd292cb7f05c31a386503f58c3
  installer                                     https://github.com/openshift/installer                                     2055609f95b19322ee6cfdd0bea73399297c4a3e
  installer-artifacts                           https://github.com/openshift/installer                                     2055609f95b19322ee6cfdd0bea73399297c4a3e
  ironic                                        https://github.com/openshift/ironic-image                                  7b39640a4517f9d0077b5d712715d997c0f44746
  ironic-hardware-inventory-recorder            https://github.com/openshift/ironic-hardware-inventory-recorder-image      5f57661eaaa3f8714c2b1904313b9ecf1a7bdba4
  ironic-inspector                              https://github.com/openshift/ironic-inspector-image                        b83630cefa1a72b2d8ce03c9701acb4f117ab6f5
  ironic-ipa-downloader                         https://github.com/openshift/ironic-ipa-downloader                         1b2dae6b0e7ce5739fbd23fc5b6501d8aa6e373c
  ironic-machine-os-downloader                  https://github.com/openshift/ironic-rhcos-downloader                       2405ba63f22db1cbda255cfccf2fc84f15faa6ee
  ironic-static-ip-manager                      https://github.com/openshift/ironic-static-ip-manager                      e33314bda6f366472a2371c5598ff2d7dba111c8
  jenkins                                       https://github.com/openshift/jenkins                                       8b61349b6d454fae4e4d6521a9a1a70cb98a54cb
  jenkins-agent-maven                           https://github.com/openshift/jenkins                                       8b61349b6d454fae4e4d6521a9a1a70cb98a54cb
  jenkins-agent-nodejs                          https://github.com/openshift/jenkins                                       8b61349b6d454fae4e4d6521a9a1a70cb98a54cb
  k8s-prometheus-adapter                        https://github.com/openshift/k8s-prometheus-adapter                        3d1c61e9a783d8c4a764b130837682fa9ab8e9df
  keepalived-ipfailover                         https://github.com/openshift/images                                        1be922de56a99f4e254eb9c2fc42d30e5596cf77
  kube-client-agent                             https://github.com/openshift/kubecsr                                       878550ca31308b77ebe4bf5574440a1457ec7fe7
  kube-etcd-signer-server                       https://github.com/openshift/kubecsr                                       878550ca31308b77ebe4bf5574440a1457ec7fe7
  kube-proxy                                    https://github.com/openshift/sdn                                           85ab10333979853fee4e607f0d0a8d87dbf8c0b2
  kube-rbac-proxy                               https://github.com/openshift/kube-rbac-proxy                               84e7dac69f8e3f90620729ee730327a3e168963d
  kube-state-metrics                            https://github.com/openshift/kube-state-metrics                            83f9fc5685cf3d7732687e1f07da6b27d6901c4f
  kuryr-cni                                     https://github.com/openshift/kuryr-kubernetes                              1673bfa4581a5286c4bb4c7a58c47b502d5f4a3a
  kuryr-controller                              https://github.com/openshift/kuryr-kubernetes                              1673bfa4581a5286c4bb4c7a58c47b502d5f4a3a
  libvirt-machine-controllers                   https://github.com/openshift/cluster-api-provider-libvirt                  dd3fc51a8b7f51da61643a6286eb0fecfa2d03cd
  local-storage-static-provisioner              https://github.com/openshift/sig-storage-local-static-provisioner          da340b475eb847c17cefe2f2f9bf9be1a3382d4a
  machine-api-operator                          https://github.com/openshift/machine-api-operator                          f552d33b6daa5b21eac7c0fde89eaa675e23e78a
  machine-config-operator                       https://github.com/openshift/machine-config-operator                       25bb6aeb58135c38a667e849edf5244871be4992
  machine-os-content                                                                                                       
  mdns-publisher                                https://github.com/openshift/mdns-publisher                                24135de29fc348f3ef455a288013cc1912d969bc
  multus-admission-controller                   https://github.com/openshift/multus-admission-controller                   ead10a3cf30c5b4a2fbd7c987d83c07a46c5d5da
  multus-cni                                    https://github.com/openshift/multus-cni                                    1cb7d0f9c0a336ba7f33a0e2800c12205f10878c
  must-gather                                   https://github.com/openshift/must-gather                                   23037ad16c0aafd7186431c93ebca3f0d426a08d
  oauth-proxy                                   https://github.com/openshift/oauth-proxy                                   fbd062bbcb23538863a735f9823ac3fd6f47429e
  oauth-server                                  https://github.com/openshift/oauth-server                                  d3065e59ce948f10364d0dd88cb70b860ec65ee2
  openshift-apiserver                           https://github.com/openshift/openshift-apiserver                           6868787f33f63c1c167c42f195a8bc98f89e827d
  openshift-controller-manager                  https://github.com/openshift/openshift-controller-manager                  72436956543f5a9587a24191d0f130e312b82e50
  openshift-state-metrics                       https://github.com/openshift/openshift-state-metrics                       c01d2de651071389d2621c46e934fd9cb2bf4b8d
  openstack-machine-controllers                 https://github.com/openshift/cluster-api-provider-openstack                c14315ec71021976371f457598351db5c00fc586
  operator-lifecycle-manager                    https://github.com/operator-framework/operator-lifecycle-manager           c74333dbe61d92e1a75b09f74c0dc2c99b05405a
  operator-marketplace                          https://github.com/operator-framework/operator-marketplace                 5a02f89919b89dac572d9b5264a22aeccc87da77
  operator-registry                             https://github.com/operator-framework/operator-registry                    4aef889d760333aebac0f01459b3a8bf698fff0b
  ovn-kubernetes                                https://github.com/openshift/ovn-kubernetes                                2808c2cbb256460d929491f33ec5858ceebb211c
  pod                                           https://github.com/openshift/images                                        1be922de56a99f4e254eb9c2fc42d30e5596cf77
  prom-label-proxy                              https://github.com/openshift/prom-label-proxy                              b8153a7f39f1a34c03059f1b9dfb05d2c1297414
  prometheus                                    https://github.com/openshift/prometheus                                    c086098da5f01ab9565bdda772d3459bea2913f2
  prometheus-alertmanager                       https://github.com/openshift/prometheus-alertmanager                       11aa2a87c5bfb84cba76b90a3867a06a55e2605e
  prometheus-config-reloader                    https://github.com/openshift/prometheus-operator                           6ba21de5746f7b009a24e48cc929135ddb3396ac
  prometheus-node-exporter                      https://github.com/openshift/node_exporter                                 1ea04fb0e09c14440bb97c3468b97f99db07dcfd
  prometheus-operator                           https://github.com/openshift/prometheus-operator                           6ba21de5746f7b009a24e48cc929135ddb3396ac
  sdn                                           https://github.com/openshift/sdn                                           85ab10333979853fee4e607f0d0a8d87dbf8c0b2
  service-ca-operator                           https://github.com/openshift/service-ca-operator                           774c394da334dec446703545d4baaf89611ccb9d
  service-catalog                               https://github.com/openshift/service-catalog                               6f75f8047dfab6ca8b979833f7f2a6036deee488
  telemeter                                     https://github.com/openshift/telemeter                                     aa7942931890dd361c117a6cbb7d84451415f7bf
  tests                                         https://github.com/openshift/ose                                           0aee6a89a377364a4c73ab1c67bfb4bb35600a59
  thanos                                        https://github.com/openshift/thanos                                        4aaa91e9b982177e113c80567ec3808dcf122769
```

We can also rsh (remote shell access) into the running Operator and see the various manifests associated with the installed release of OpenShift:

```
$ oc rsh -n openshift-cluster-version deployments/cluster-version-operator
sh-4.2#
```

Then to list the available manifests:

```
sh-4.2# ls -l /release-manifests/
total 2648
-r--r--r--. 1 root root   6942 Jan 21 19:32 0000_00_cluster-version-operator_01_clusteroperator.crd.yaml
-r--r--r--. 1 root root  16818 Jan 21 19:32 0000_00_cluster-version-operator_01_clusterversion.crd.yaml
-r--r--r--. 1 root root  10252 Jan 21 19:32 0000_03_authorization-openshift_01_rolebindingrestriction.crd.yaml
-r--r--r--. 1 root root   4549 Jan 21 19:32 0000_03_config-operator_01_operatorhub.crd.yaml
-r--r--r--. 1 root root   4422 Jan 21 19:32 0000_03_config-operator_01_proxy.crd.yaml
-r--r--r--. 1 root root  10443 Jan 21 19:32 0000_03_quota-openshift_01_clusterresourcequota.crd.yaml
-r--r--r--. 1 root root  14139 Jan 21 19:32 0000_03_security-openshift_01_scc.crd.yaml
-r--r--r--. 1 root root    146 Jan 21 19:32 0000_05_config-operator_02_apiserver.cr.yaml
-r--r--r--. 1 root root    151 Jan 21 19:32 0000_05_config-operator_02_authentication.cr.yaml
-r--r--r--. 1 root root    142 Jan 21 19:32 0000_05_config-operator_02_build.cr.yaml
-r--r--r--. 1 root root    144 Jan 21 19:32 0000_05_config-operator_02_console.cr.yaml
-r--r--r--. 1 root root    140 Jan 21 19:32 0000_05_config-operator_02_dns.cr.yaml
-r--r--r--. 1 root root    148 Jan 21 19:32 0000_05_config-operator_02_featuregate.cr.yaml
-r--r--r--. 1 root root    142 Jan 21 19:32 0000_05_config-operator_02_image.cr.yaml
-r--r--r--. 1 root root    151 Jan 21 19:32 0000_05_config-operator_02_infrastructure.cr.yaml
-r--r--r--. 1 root root    144 Jan 21 19:32 0000_05_config-operator_02_ingress.cr.yaml
-r--r--r--. 1 root root    144 Jan 21 19:32 0000_05_config-operator_02_network.cr.yaml
-r--r--r--. 1 root root    142 Jan 21 19:32 0000_05_config-operator_02_oauth.cr.yaml
-r--r--r--. 1 root root    148 Jan 21 19:32 0000_05_config-operator_02_operatorhub.cr.yaml
-r--r--r--. 1 root root    144 Jan 21 19:32 0000_05_config-operator_02_project.cr.yaml
-r--r--r--. 1 root root    142 Jan 21 19:32 0000_05_config-operator_02_proxy.cr.yaml
-r--r--r--. 1 root root    146 Jan 21 19:32 0000_05_config-operator_02_scheduler.cr.yaml
-r--r--r--. 1 root root  12652 Jan 21 19:32 0000_10_config-operator_01_apiserver.crd.yaml
-r--r--r--. 1 root root   6359 Jan 21 19:32 0000_10_config-operator_01_authentication.crd.yaml
-r--r--r--. 1 root root  20067 Jan 21 19:32 0000_10_config-operator_01_build.crd.yaml
-r--r--r--. 1 root root   3111 Jan 21 19:32 0000_10_config-operator_01_console.crd.yaml
-r--r--r--. 1 root root   5225 Jan 21 19:32 0000_10_config-operator_01_dns.crd.yaml
-r--r--r--. 1 root root   3146 Jan 21 19:32 0000_10_config-operator_01_featuregate.crd.yaml
-r--r--r--. 1 root root   7188 Jan 21 19:32 0000_10_config-operator_01_image.crd.yaml
-r--r--r--. 1 root root   4716 Jan 21 19:32 0000_10_config-operator_01_imagecontentsourcepolicy.crd.yaml
-r--r--r--. 1 root root  12305 Jan 21 19:32 0000_10_config-operator_01_infrastructure.crd.yaml
-r--r--r--. 1 root root   2298 Jan 21 19:32 0000_10_config-operator_01_ingress.crd.yaml
-r--r--r--. 1 root root   6489 Jan 21 19:32 0000_10_config-operator_01_network.crd.yaml
-r--r--r--. 1 root root  37015 Jan 21 19:32 0000_10_config-operator_01_oauth.crd.yaml
-r--r--r--. 1 root root    166 Jan 21 19:32 0000_10_config-operator_01_openshift-config-managed-ns.yaml
-r--r--r--. 1 root root    158 Jan 21 19:32 0000_10_config-operator_01_openshift-config-ns.yaml
-r--r--r--. 1 root root   2509 Jan 21 19:32 0000_10_config-operator_01_project.crd.yaml
-r--r--r--. 1 root root   4478 Jan 21 19:32 0000_10_config-operator_01_scheduler.crd.yaml
-r--r--r--. 1 root root    538 Jan 21 19:32 0000_10_config-operator_02_config.clusterrole.yaml
-r--r--r--. 1 root root   3056 Jan 21 19:28 0000_10_consoleclidownload.crd.yaml
-r--r--r--. 1 root root   4054 Jan 21 19:28 0000_10_consoleexternalloglink.crd.yaml
-r--r--r--. 1 root root   6917 Jan 21 19:28 0000_10_consolelink.crd.yaml
-r--r--r--. 1 root root   3254 Jan 21 19:28 0000_10_consolenotification.crd.yaml
-r--r--r--. 1 root root   3778 Jan 21 19:28 0000_10_consoleyamlsample.crd.yaml
-r--r--r--. 1 root root   8519 Jan 21 19:15 0000_10_samplesconfig.crd.yaml
-r--r--r--. 1 root root    219 Jan 21 19:21 0000_20_kube-apiserver-operator_00_namespace.yaml
-r--r--r--. 1 root root   9131 Jan 21 19:21 0000_20_kube-apiserver-operator_01_config.crd.yaml
-r--r--r--. 1 root root    176 Jan 21 19:21 0000_20_kube-apiserver-operator_01_operator.cr.yaml
-r--r--r--. 1 root root    492 Jan 21 19:21 0000_20_kube-apiserver-operator_02_service.yaml
-r--r--r--. 1 root root    223 Jan 21 19:21 0000_20_kube-apiserver-operator_03_configmap.yaml
-r--r--r--. 1 root root    297 Jan 21 19:21 0000_20_kube-apiserver-operator_04_clusterrolebinding.yaml
-r--r--r--. 1 root root    169 Jan 21 19:21 0000_20_kube-apiserver-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root   2614 Jan 21 19:21 0000_20_kube-apiserver-operator_06_deployment.yaml
-r--r--r--. 1 root root    332 Jan 21 19:21 0000_20_kube-apiserver-operator_07_clusteroperator.yaml
-r--r--r--. 1 root root    228 Jan 21 19:21 0000_25_kube-controller-manager-operator_00_namespace.yaml
-r--r--r--. 1 root root   9232 Jan 21 19:21 0000_25_kube-controller-manager-operator_01_config.crd.yaml
-r--r--r--. 1 root root    183 Jan 21 19:21 0000_25_kube-controller-manager-operator_01_operator.cr.yaml
-r--r--r--. 1 root root    461 Jan 21 19:21 0000_25_kube-controller-manager-operator_02_service.yaml
-r--r--r--. 1 root root    241 Jan 21 19:21 0000_25_kube-controller-manager-operator_03_configmap.yaml
-r--r--r--. 1 root root    324 Jan 21 19:21 0000_25_kube-controller-manager-operator_04_clusterrolebinding.yaml
-r--r--r--. 1 root root    143 Jan 21 19:21 0000_25_kube-controller-manager-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root   2897 Jan 21 19:21 0000_25_kube-controller-manager-operator_06_deployment.yaml
-r--r--r--. 1 root root    362 Jan 21 19:21 0000_25_kube-controller-manager-operator_07_clusteroperator.yaml
-r--r--r--. 1 root root    219 Jan 21 19:45 0000_25_kube-scheduler-operator_00_namespace.yaml
-r--r--r--. 1 root root   9160 Jan 21 19:45 0000_25_kube-scheduler-operator_01_config.crd.yaml
-r--r--r--. 1 root root    176 Jan 21 19:45 0000_25_kube-scheduler-operator_02_operator.cr.yaml
-r--r--r--. 1 root root    512 Jan 21 19:45 0000_25_kube-scheduler-operator_02_service.yaml
-r--r--r--. 1 root root    233 Jan 21 19:45 0000_25_kube-scheduler-operator_03_configmap.yaml
-r--r--r--. 1 root root    315 Jan 21 19:45 0000_25_kube-scheduler-operator_04_clusterrolebinding.yaml
-r--r--r--. 1 root root    188 Jan 21 19:45 0000_25_kube-scheduler-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root   2575 Jan 21 19:45 0000_25_kube-scheduler-operator_06_deployment.yaml
-r--r--r--. 1 root root    332 Jan 21 19:45 0000_25_kube-scheduler-operator_07_clusteroperator.yaml
-r--r--r--. 1 root root   2615 Jan 21 19:28 0000_30_machine-api-operator_00_credentials-request.yaml
-r--r--r--. 1 root root    326 Jan 21 19:28 0000_30_machine-api-operator_00_namespace.yaml
-r--r--r--. 1 root root   2166 Jan 21 19:28 0000_30_machine-api-operator_01_images.configmap.yaml
-r--r--r--. 1 root root  13852 Jan 21 19:28 0000_30_machine-api-operator_02_machine.crd.yaml
-r--r--r--. 1 root root  13661 Jan 21 19:28 0000_30_machine-api-operator_03_machineset.crd.yaml
-r--r--r--. 1 root root   5935 Jan 21 19:28 0000_30_machine-api-operator_07_machinehealthcheck.crd.yaml
-r--r--r--. 1 root root  14108 Jan 21 19:28 0000_30_machine-api-operator_08_baremetalhost.crd.yaml
-r--r--r--. 1 root root   5946 Jan 21 19:28 0000_30_machine-api-operator_09_rbac.yaml
-r--r--r--. 1 root root    296 Jan 21 19:28 0000_30_machine-api-operator_10_kube-rbac-proxy-config.yaml
-r--r--r--. 1 root root    471 Jan 21 19:28 0000_30_machine-api-operator_10_service.yaml
-r--r--r--. 1 root root   3025 Jan 21 19:28 0000_30_machine-api-operator_11_deployment.yaml
-r--r--r--. 1 root root    242 Jan 21 19:28 0000_30_machine-api-operator_12_clusteroperator.yaml
-r--r--r--. 1 root root    214 Jan 21 19:28 0000_30_openshift-apiserver-operator_00_namespace.yaml
-r--r--r--. 1 root root   6542 Jan 21 19:28 0000_30_openshift-apiserver-operator_01_config.crd.yaml
-r--r--r--. 1 root root    180 Jan 21 19:28 0000_30_openshift-apiserver-operator_01_operator.cr.yaml
-r--r--r--. 1 root root    223 Jan 21 19:28 0000_30_openshift-apiserver-operator_03_configmap.yaml
-r--r--r--. 1 root root    235 Jan 21 19:28 0000_30_openshift-apiserver-operator_03_trusted_ca_cm.yaml
-r--r--r--. 1 root root    302 Jan 21 19:28 0000_30_openshift-apiserver-operator_04_roles.yaml
-r--r--r--. 1 root root    173 Jan 21 19:28 0000_30_openshift-apiserver-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root    412 Jan 21 19:28 0000_30_openshift-apiserver-operator_06_service.yaml
-r--r--r--. 1 root root   2549 Jan 21 19:28 0000_30_openshift-apiserver-operator_07_deployment.yaml
-r--r--r--. 1 root root    250 Jan 21 19:28 0000_30_openshift-apiserver-operator_08_clusteroperator.yaml
-r--r--r--. 1 root root    337 Jan 21 19:28 0000_50_cloud-credential-operator_00_clusterreader_clusterrole.yaml
-r--r--r--. 1 root root    256 Jan 21 19:28 0000_50_cloud-credential-operator_00_namespace.yaml
-r--r--r--. 1 root root   4129 Jan 21 19:28 0000_50_cloud-credential-operator_00_v1_crd.yaml
-r--r--r--. 1 root root    217 Jan 21 19:28 0000_50_cloud-credential-operator_01_operator_configmap.yaml
-r--r--r--. 1 root root   7520 Jan 21 19:28 0000_50_cloud-credential-operator_05_deployment.yaml
-r--r--r--. 1 root root    658 Jan 21 19:28 0000_50_cloud-credential-operator_07_cred-iam-ro.yaml
-r--r--r--. 1 root root    235 Jan 21 19:28 0000_50_cloud-credential-operator_10_cluster-operator.yaml
-r--r--r--. 1 root root    369 Jan 21 19:31 0000_50_cluster-authentication-operator_00_namespace.yaml
-r--r--r--. 1 root root   5947 Jan 21 19:31 0000_50_cluster-authentication-operator_01_config.crd.yaml
-r--r--r--. 1 root root    177 Jan 21 19:31 0000_50_cluster-authentication-operator_02_config.cr.yaml
-r--r--r--. 1 root root    427 Jan 21 19:31 0000_50_cluster-authentication-operator_02_service.yaml
-r--r--r--. 1 root root    280 Jan 21 19:31 0000_50_cluster-authentication-operator_03_configmap.yaml
-r--r--r--. 1 root root    250 Jan 21 19:31 0000_50_cluster-authentication-operator_03_operand_trusted_ca.yaml
-r--r--r--. 1 root root    240 Jan 21 19:31 0000_50_cluster-authentication-operator_03_operator_trusted_ca.yaml
-r--r--r--. 1 root root    402 Jan 21 19:31 0000_50_cluster-authentication-operator_04_roles.yaml
-r--r--r--. 1 root root    315 Jan 21 19:31 0000_50_cluster-authentication-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root 638029 Jan 21 19:31 0000_50_cluster-authentication-operator_06_branding_secret.yaml
-r--r--r--. 1 root root   2966 Jan 21 19:31 0000_50_cluster-authentication-operator_07_deployment.yaml
-r--r--r--. 1 root root    289 Jan 21 19:31 0000_50_cluster-authentication-operator_08_clusteroperator.yaml
-r--r--r--. 1 root root    279 Jan 21 19:28 0000_50_cluster-autoscaler-operator_00_namespace.yaml
-r--r--r--. 1 root root   6833 Jan 21 19:28 0000_50_cluster-autoscaler-operator_01_clusterautoscaler.crd.yaml
-r--r--r--. 1 root root   5567 Jan 21 19:28 0000_50_cluster-autoscaler-operator_02_machineautoscaler.crd.yaml
-r--r--r--. 1 root root   6006 Jan 21 19:28 0000_50_cluster-autoscaler-operator_03_rbac.yaml
-r--r--r--. 1 root root    590 Jan 21 19:28 0000_50_cluster-autoscaler-operator_04_service.yaml
-r--r--r--. 1 root root    245 Jan 21 19:28 0000_50_cluster-autoscaler-operator_05_configmap.yaml
-r--r--r--. 1 root root    323 Jan 21 19:28 0000_50_cluster-autoscaler-operator_06_kube_rbac_proxy.yaml
-r--r--r--. 1 root root    738 Jan 21 19:28 0000_50_cluster-autoscaler-operator_06_servicemonitor.yaml
-r--r--r--. 1 root root   3811 Jan 21 19:28 0000_50_cluster-autoscaler-operator_07_deployment.yaml
-r--r--r--. 1 root root    236 Jan 21 19:28 0000_50_cluster-autoscaler-operator_08_clusteroperator.yaml
-r--r--r--. 1 root root    587 Jan 21 19:28 0000_50_cluster-autoscaler-operator_09_alertrules.yaml
-r--r--r--. 1 root root  26161 Jan 21 19:25 0000_50_cluster-image-registry-operator_00-crd.yaml
-r--r--r--. 1 root root    217 Jan 21 19:25 0000_50_cluster-image-registry-operator_01-namespace.yaml
-r--r--r--. 1 root root    443 Jan 21 19:25 0000_50_cluster-image-registry-operator_01-registry-credentials-request-azure.yaml
-r--r--r--. 1 root root    506 Jan 21 19:25 0000_50_cluster-image-registry-operator_01-registry-credentials-request-gcs.yaml
-r--r--r--. 1 root root    409 Jan 21 19:25 0000_50_cluster-image-registry-operator_01-registry-credentials-request-openstack.yaml
-r--r--r--. 1 root root   1008 Jan 21 19:25 0000_50_cluster-image-registry-operator_01-registry-credentials-request.yaml
-r--r--r--. 1 root root   1865 Jan 21 19:25 0000_50_cluster-image-registry-operator_02-rbac.yaml
-r--r--r--. 1 root root    128 Jan 21 19:25 0000_50_cluster-image-registry-operator_03-sa.yaml
-r--r--r--. 1 root root    228 Jan 21 19:25 0000_50_cluster-image-registry-operator_04-ca-trusted.yaml
-r--r--r--. 1 root root    549 Jan 21 19:25 0000_50_cluster-image-registry-operator_05-ca-rbac.yaml
-r--r--r--. 1 root root    104 Jan 21 19:25 0000_50_cluster-image-registry-operator_06-ca-serviceaccount.yaml
-r--r--r--. 1 root root    400 Jan 21 19:25 0000_50_cluster-image-registry-operator_07-operator-service.yaml
-r--r--r--. 1 root root   3386 Jan 21 19:25 0000_50_cluster-image-registry-operator_07-operator.yaml
-r--r--r--. 1 root root    162 Jan 21 19:25 0000_50_cluster-image-registry-operator_08-clusteroperator.yaml
-r--r--r--. 1 root root   1128 Jan 21 19:25 0000_50_cluster-image-registry-operator_09-prometheus-rules.yaml
-r--r--r--. 1 root root   2305 Jan 21 19:26 0000_50_cluster-ingress-operator_00-cluster-role.yaml
-r--r--r--. 1 root root   4452 Jan 21 19:26 0000_50_cluster-ingress-operator_00-custom-resource-definition-internal.yaml
-r--r--r--. 1 root root  31552 Jan 21 19:26 0000_50_cluster-ingress-operator_00-custom-resource-definition.yaml
-r--r--r--. 1 root root   1464 Jan 21 19:26 0000_50_cluster-ingress-operator_00-ingress-credentials-request.yaml
-r--r--r--. 1 root root    266 Jan 21 19:26 0000_50_cluster-ingress-operator_00-namespace.yaml
-r--r--r--. 1 root root    369 Jan 21 19:26 0000_50_cluster-ingress-operator_01-cluster-role-binding.yaml
-r--r--r--. 1 root root    367 Jan 21 19:26 0000_50_cluster-ingress-operator_01-role-binding.yaml
-r--r--r--. 1 root root    477 Jan 21 19:26 0000_50_cluster-ingress-operator_01-role.yaml
-r--r--r--. 1 root root    196 Jan 21 19:26 0000_50_cluster-ingress-operator_01-service-account.yaml
-r--r--r--. 1 root root    344 Jan 21 19:26 0000_50_cluster-ingress-operator_01-service.yaml
-r--r--r--. 1 root root    323 Jan 21 19:26 0000_50_cluster-ingress-operator_01-trusted-ca-configmap.yaml
-r--r--r--. 1 root root   3046 Jan 21 19:26 0000_50_cluster-ingress-operator_02-deployment.yaml
-r--r--r--. 1 root root    348 Jan 21 19:26 0000_50_cluster-ingress-operator_03-cluster-operator.yaml
-r--r--r--. 1 root root    307 Jan 21 19:31 0000_50_cluster-machine-approver_00-ns.yaml
-r--r--r--. 1 root root   1767 Jan 21 19:31 0000_50_cluster-machine-approver_01-rbac.yaml
-r--r--r--. 1 root root    325 Jan 21 19:31 0000_50_cluster-machine-approver_02-kube-rbac-proxy-config.yaml
-r--r--r--. 1 root root    407 Jan 21 19:31 0000_50_cluster-machine-approver_03-metrics-service.yaml
-r--r--r--. 1 root root   3306 Jan 21 19:31 0000_50_cluster-machine-approver_04-deployment.yaml
-r--r--r--. 1 root root    146 Jan 21 19:31 0000_50_cluster-node-tuning-operator_01-namespace.yaml
-r--r--r--. 1 root root   5655 Jan 21 19:31 0000_50_cluster-node-tuning-operator_02-crd.yaml
-r--r--r--. 1 root root   1900 Jan 21 19:31 0000_50_cluster-node-tuning-operator_03-rbac.yaml
-r--r--r--. 1 root root   2191 Jan 21 19:31 0000_50_cluster-node-tuning-operator_04-operator.yaml
-r--r--r--. 1 root root    159 Jan 21 19:31 0000_50_cluster-node-tuning-operator_05-clusteroperator.yaml
-r--r--r--. 1 root root    223 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_00_namespace.yaml
-r--r--r--. 1 root root    214 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_01_operand-namespace.yaml
-r--r--r--. 1 root root   6057 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_02_config.crd.yaml
-r--r--r--. 1 root root    189 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_03_config.cr.yaml
-r--r--r--. 1 root root    241 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_03_configmap.yaml
-r--r--r--. 1 root root    538 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_04_metricservice.yaml
-r--r--r--. 1 root root    413 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_05_builder-deployer-config.yaml
-r--r--r--. 1 root root    329 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_06_roles.yaml
-r--r--r--. 1 root root    200 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_07_serviceaccount.yaml
-r--r--r--. 1 root root   2528 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_09_deployment.yaml
-r--r--r--. 1 root root    259 Jan 21 19:28 0000_50_cluster-openshift-controller-manager-operator_10_clusteroperator.yaml
-r--r--r--. 1 root root    223 Jan 21 19:15 0000_50_cluster-samples-operator_01-namespace.yaml
-r--r--r--. 1 root root   1892 Jan 21 19:15 0000_50_cluster-samples-operator_010-prometheus-rules.yaml
-r--r--r--. 1 root root    131 Jan 21 19:15 0000_50_cluster-samples-operator_02-sa.yaml
-r--r--r--. 1 root root   1722 Jan 21 19:15 0000_50_cluster-samples-operator_03-rbac.yaml
-r--r--r--. 1 root root    334 Jan 21 19:15 0000_50_cluster-samples-operator_04-openshift-rbac.yaml
-r--r--r--. 1 root root    591 Jan 21 19:15 0000_50_cluster-samples-operator_05-kube-system-rbac.yaml
-r--r--r--. 1 root root    379 Jan 21 19:15 0000_50_cluster-samples-operator_06-metricsservice.yaml
-r--r--r--. 1 root root   3098 Jan 21 19:15 0000_50_cluster-samples-operator_06-operator.yaml
-r--r--r--. 1 root root    456 Jan 21 19:15 0000_50_cluster-samples-operator_06-servicemonitor.yaml
-r--r--r--. 1 root root    166 Jan 21 19:15 0000_50_cluster-samples-operator_07-clusteroperator.yaml
-r--r--r--. 1 root root   2216 Jan 21 19:15 0000_50_cluster-samples-operator_08-openshift-imagestreams.yaml
-r--r--r--. 1 root root    565 Jan 21 19:15 0000_50_cluster-samples-operator_09-servicemonitor-rbac.yaml
-r--r--r--. 1 root root    134 Jan 21 19:24 0000_50_cluster-storage-operator_00-namespace.yaml
-r--r--r--. 1 root root    323 Jan 21 19:24 0000_50_cluster-storage-operator_01-cluster-role-binding.yaml
-r--r--r--. 1 root root    574 Jan 21 19:24 0000_50_cluster-storage-operator_01-cluster-role.yaml
-r--r--r--. 1 root root    309 Jan 21 19:24 0000_50_cluster-storage-operator_01-role-binding.yaml
-r--r--r--. 1 root root    493 Jan 21 19:24 0000_50_cluster-storage-operator_01-role.yaml
-r--r--r--. 1 root root    127 Jan 21 19:24 0000_50_cluster-storage-operator_01-service-account.yaml
-r--r--r--. 1 root root   1984 Jan 21 19:24 0000_50_cluster-storage-operator_02-deployment.yaml
-r--r--r--. 1 root root    143 Jan 21 19:24 0000_50_cluster-storage-operator_03-cluster-operator.yaml
-r--r--r--. 1 root root    208 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_00_namespace.yaml
-r--r--r--. 1 root root   6039 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_02_config.crd.yaml
-r--r--r--. 1 root root    205 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_03_config.cr.yaml
-r--r--r--. 1 root root    255 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_03_configmap.yaml
-r--r--r--. 1 root root    262 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_03_version-configmap.yaml
-r--r--r--. 1 root root    350 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_04_roles.yaml
-r--r--r--. 1 root root    221 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_05_serviceaccount.yaml
-r--r--r--. 1 root root    476 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_06_service.yaml
-r--r--r--. 1 root root   2381 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_07_deployment.yaml
-r--r--r--. 1 root root    174 Jan 21 19:27 0000_50_cluster-svcat-apiserver-operator_08_cluster-operator.yaml
-r--r--r--. 1 root root    207 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_00_namespace.yaml
-r--r--r--. 1 root root   6087 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_02_config.crd.yaml
-r--r--r--. 1 root root    215 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_03_config.cr.yaml
-r--r--r--. 1 root root    273 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_03_configmap.yaml
-r--r--r--. 1 root root    535 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_04_metricservice.yaml
-r--r--r--. 1 root root    377 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_06_roles.yaml
-r--r--r--. 1 root root    248 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_07_serviceaccount.yaml
-r--r--r--. 1 root root   2583 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_09_deployment.yaml
-r--r--r--. 1 root root    182 Jan 21 19:39 0000_50_cluster-svcat-controller-manager-operator_10_cluster-operator.yaml
-r--r--r--. 1 root root    542 Jan 21 19:21 0000_50_cluster_monitoring_operator_01-namespace.yaml
-r--r--r--. 1 root root   6108 Jan 21 19:21 0000_50_cluster_monitoring_operator_02-role.yaml
-r--r--r--. 1 root root    447 Jan 21 19:21 0000_50_cluster_monitoring_operator_03-role-binding.yaml
-r--r--r--. 1 root root   4114 Jan 21 19:21 0000_50_cluster_monitoring_operator_04-deployment.yaml
-r--r--r--. 1 root root    155 Jan 21 19:21 0000_50_cluster_monitoring_operator_05-clusteroperator.yaml
-r--r--r--. 1 root root    187 Jan 21 19:28 0000_50_console-operator_01-oauth.yaml
-r--r--r--. 1 root root    170 Jan 21 19:28 0000_50_console-operator_01-operator-config.yaml
-r--r--r--. 1 root root    301 Jan 21 19:28 0000_50_console-operator_02-namespace.yaml
-r--r--r--. 1 root root    389 Jan 21 19:28 0000_50_console-operator_03-rbac-role-cluster-extensions.yaml
-r--r--r--. 1 root root    946 Jan 21 19:28 0000_50_console-operator_03-rbac-role-cluster.yaml
-r--r--r--. 1 root root    923 Jan 21 19:28 0000_50_console-operator_03-rbac-role-ns-console.yaml
-r--r--r--. 1 root root    555 Jan 21 19:28 0000_50_console-operator_03-rbac-role-ns-openshift-config-managed.yaml
-r--r--r--. 1 root root    234 Jan 21 19:28 0000_50_console-operator_03-rbac-role-ns-openshift-config.yaml
-r--r--r--. 1 root root    453 Jan 21 19:28 0000_50_console-operator_03-rbac-role-ns-operator.yaml
-r--r--r--. 1 root root   1106 Jan 21 19:28 0000_50_console-operator_04-rbac-rolebinding-cluster.yaml
-r--r--r--. 1 root root   1883 Jan 21 19:28 0000_50_console-operator_04-rbac-rolebinding.yaml
-r--r--r--. 1 root root    164 Jan 21 19:28 0000_50_console-operator_05-config-public.yaml
-r--r--r--. 1 root root    290 Jan 21 19:28 0000_50_console-operator_05-config.yaml
-r--r--r--. 1 root root    416 Jan 21 19:28 0000_50_console-operator_05-service.yaml
-r--r--r--. 1 root root    111 Jan 21 19:28 0000_50_console-operator_06-sa.yaml
-r--r--r--. 1 root root   4566 Jan 21 19:28 0000_50_console-operator_07-downloads-deployment.yaml
-r--r--r--. 1 root root    540 Jan 21 19:28 0000_50_console-operator_07-downloads-helm.yaml
-r--r--r--. 1 root root    280 Jan 21 19:28 0000_50_console-operator_07-downloads-route.yaml
-r--r--r--. 1 root root    224 Jan 21 19:28 0000_50_console-operator_07-downloads-service.yaml
-r--r--r--. 1 root root   2406 Jan 21 19:28 0000_50_console-operator_07-operator.yaml
-r--r--r--. 1 root root    156 Jan 21 19:28 0000_50_console-operator_08-clusteroperator.yaml
-r--r--r--. 1 root root    183 Jan 21 19:21 0000_50_insights-operator_02-namespace.yaml
-r--r--r--. 1 root root   3515 Jan 21 19:21 0000_50_insights-operator_03-clusterrole.yaml
-r--r--r--. 1 root root    229 Jan 21 19:21 0000_50_insights-operator_03-prometheus_role.yaml
-r--r--r--. 1 root root    297 Jan 21 19:21 0000_50_insights-operator_03-prometheus_rolebinding.yaml
-r--r--r--. 1 root root    224 Jan 21 19:21 0000_50_insights-operator_04-proxy-cert-configmap.yaml
-r--r--r--. 1 root root    192 Jan 21 19:21 0000_50_insights-operator_04-serviceaccount.yaml
-r--r--r--. 1 root root    436 Jan 21 19:21 0000_50_insights-operator_05-service.yaml
-r--r--r--. 1 root root   2405 Jan 21 19:21 0000_50_insights-operator_06-deployment.yaml
-r--r--r--. 1 root root    235 Jan 21 19:21 0000_50_insights-operator_07-cluster-operator.yaml
-r--r--r--. 1 root root    769 Jan 21 19:21 0000_50_insights-operator_07-servicemonitor.yaml
-r--r--r--. 1 root root    399 Jan 21 19:28 0000_50_olm_00-namespace.yaml
-r--r--r--. 1 root root    743 Jan 21 19:28 0000_50_olm_01-olm-operator.serviceaccount.yaml
-r--r--r--. 1 root root    814 Jan 21 19:28 0000_50_olm_02-bastion.yaml
-r--r--r--. 1 root root  35382 Jan 21 19:28 0000_50_olm_03-clusterserviceversion.crd.yaml
-r--r--r--. 1 root root   2095 Jan 21 19:28 0000_50_olm_04-installplan.crd.yaml
-r--r--r--. 1 root root  91995 Jan 21 19:28 0000_50_olm_05-subscription.crd.yaml
-r--r--r--. 1 root root   4403 Jan 21 19:28 0000_50_olm_06-catalogsource.crd.yaml
-r--r--r--. 1 root root   2608 Jan 21 19:28 0000_50_olm_07-olm-operator.deployment.yaml
-r--r--r--. 1 root root   2513 Jan 21 19:28 0000_50_olm_08-catalog-operator.deployment.yaml
-r--r--r--. 1 root root   1245 Jan 21 19:28 0000_50_olm_09-aggregated.clusterrole.yaml
-r--r--r--. 1 root root   3269 Jan 21 19:28 0000_50_olm_10-operatorgroup.crd.yaml
-r--r--r--. 1 root root    335 Jan 21 19:28 0000_50_olm_13-operatorgroup-default.yaml
-r--r--r--. 1 root root   4256 Jan 21 19:28 0000_50_olm_15-packageserver.clusterserviceversion.yaml
-r--r--r--. 1 root root    529 Jan 21 19:28 0000_50_olm_99-operatorstatus.yaml
-r--r--r--. 1 root root    189 Jan 21 19:22 0000_50_operator-marketplace_01_namespace.yaml
-r--r--r--. 1 root root   4502 Jan 21 19:22 0000_50_operator-marketplace_02_catalogsourceconfig.crd.yaml
-r--r--r--. 1 root root   5616 Jan 21 19:22 0000_50_operator-marketplace_03_operatorsource.crd.yaml
-r--r--r--. 1 root root    110 Jan 21 19:22 0000_50_operator-marketplace_04_service_account.yaml
-r--r--r--. 1 root root   1750 Jan 21 19:22 0000_50_operator-marketplace_05_role.yaml
-r--r--r--. 1 root root    631 Jan 21 19:22 0000_50_operator-marketplace_06_role_binding.yaml
-r--r--r--. 1 root root    279 Jan 21 19:22 0000_50_operator-marketplace_07_configmap.yaml
-r--r--r--. 1 root root    282 Jan 21 19:22 0000_50_operator-marketplace_08_service.yaml
-r--r--r--. 1 root root   2679 Jan 21 19:22 0000_50_operator-marketplace_09_operator.yaml
-r--r--r--. 1 root root    271 Jan 21 19:22 0000_50_operator-marketplace_10_clusteroperator.yaml
-r--r--r--. 1 root root    922 Jan 21 19:22 0000_50_operator-marketplace_11_service_monitor.yaml
-r--r--r--. 1 root root   3976 Jan 21 19:22 0000_50_operator-marketplace_12_prometheus_rule.yaml
-r--r--r--. 1 root root    285 Jan 21 19:31 0000_50_service-ca-operator_00_roles.yaml
-r--r--r--. 1 root root    171 Jan 21 19:31 0000_50_service-ca-operator_01_namespace.yaml
-r--r--r--. 1 root root   6118 Jan 21 19:31 0000_50_service-ca-operator_02_crd.yaml
-r--r--r--. 1 root root    230 Jan 21 19:31 0000_50_service-ca-operator_03_cm.yaml
-r--r--r--. 1 root root    172 Jan 21 19:31 0000_50_service-ca-operator_03_operator.cr.yaml
-r--r--r--. 1 root root    156 Jan 21 19:31 0000_50_service-ca-operator_04_sa.yaml
-r--r--r--. 1 root root   1936 Jan 21 19:31 0000_50_service-ca-operator_05_deploy.yaml
-r--r--r--. 1 root root    146 Jan 21 19:31 0000_50_service-ca-operator_07_clusteroperator.yaml
-r--r--r--. 1 root root    205 Jan 21 19:26 0000_70_cluster-network-operator_00_namespace.yaml
-r--r--r--. 1 root root   5249 Jan 21 19:26 0000_70_cluster-network-operator_01_crd.yaml
-r--r--r--. 1 root root    394 Jan 21 19:26 0000_70_cluster-network-operator_01_credentialsrequest.yaml
-r--r--r--. 1 root root   3197 Jan 21 19:26 0000_70_cluster-network-operator_01_pki_crd.yaml
-r--r--r--. 1 root root    308 Jan 21 19:26 0000_70_cluster-network-operator_02_rbac.yaml
-r--r--r--. 1 root root   3092 Jan 21 19:26 0000_70_cluster-network-operator_03_deployment.yaml
-r--r--r--. 1 root root    143 Jan 21 19:26 0000_70_cluster-network-operator_04_clusteroperator.yaml
-r--r--r--. 1 root root   9434 Jan 21 19:28 0000_70_console-operator.crd.yaml
-r--r--r--. 1 root root   1208 Jan 21 19:20 0000_70_dns-operator_00-cluster-role.yaml
-r--r--r--. 1 root root   6871 Jan 21 19:20 0000_70_dns-operator_00-custom-resource-definition.yaml
-r--r--r--. 1 root root    378 Jan 21 19:20 0000_70_dns-operator_00-namespace.yaml
-r--r--r--. 1 root root    353 Jan 21 19:20 0000_70_dns-operator_01-cluster-role-binding.yaml
-r--r--r--. 1 root root    347 Jan 21 19:20 0000_70_dns-operator_01-role-binding.yaml
-r--r--r--. 1 root root    351 Jan 21 19:20 0000_70_dns-operator_01-role.yaml
-r--r--r--. 1 root root    188 Jan 21 19:20 0000_70_dns-operator_01-service-account.yaml
-r--r--r--. 1 root root    332 Jan 21 19:20 0000_70_dns-operator_01-service.yaml
-r--r--r--. 1 root root   2620 Jan 21 19:20 0000_70_dns-operator_02-deployment.yaml
-r--r--r--. 1 root root    339 Jan 21 19:20 0000_70_dns-operator_03-cluster-operator.yaml
-r--r--r--. 1 root root    465 Jan 21 19:38 0000_80_machine-config-operator_00_clusterreader_clusterrole.yaml
-r--r--r--. 1 root root    864 Jan 21 19:38 0000_80_machine-config-operator_00_namespace.yaml
-r--r--r--. 1 root root    373 Jan 21 19:38 0000_80_machine-config-operator_00_service.yaml
-r--r--r--. 1 root root    948 Jan 21 19:38 0000_80_machine-config-operator_01_mcoconfig.crd.yaml
-r--r--r--. 1 root root   1660 Jan 21 19:38 0000_80_machine-config-operator_02_images.configmap.yaml
-r--r--r--. 1 root root   1039 Jan 21 19:38 0000_80_machine-config-operator_03_rbac.yaml
-r--r--r--. 1 root root   2003 Jan 21 19:38 0000_80_machine-config-operator_04_deployment.yaml
-r--r--r--. 1 root root    418 Jan 21 19:38 0000_80_machine-config-operator_05_osimageurl.yaml
-r--r--r--. 1 root root    245 Jan 21 19:38 0000_80_machine-config-operator_06_clusteroperator.yaml
-r--r--r--. 1 root root   3004 Jan 21 19:38 0000_80_machine-config-operator_07_etcdquorumguard_deployment.yaml
-r--r--r--. 1 root root    306 Jan 21 19:38 0000_80_machine-config-operator_08_etcdquorumguard_pdb.yaml
-r--r--r--. 1 root root   1394 Jan 21 19:31 0000_90_cluster-authentication-operator_01_prometheusrbac.yaml
-r--r--r--. 1 root root   1485 Jan 21 19:31 0000_90_cluster-authentication-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root   1071 Jan 21 19:25 0000_90_cluster-image-registry-operator_00_servicemonitor-rbac.yaml
-r--r--r--. 1 root root    576 Jan 21 19:25 0000_90_cluster-image-registry-operator_01_operand-servicemonitor.yaml
-r--r--r--. 1 root root    450 Jan 21 19:25 0000_90_cluster-image-registry-operator_02_operator-servicemonitor.yaml
-r--r--r--. 1 root root    317 Jan 21 19:31 0000_90_cluster-machine-approver_01_prometheusrole.yaml
-r--r--r--. 1 root root    313 Jan 21 19:31 0000_90_cluster-machine-approver_02_prometheusrolebinding.yaml
-r--r--r--. 1 root root    649 Jan 21 19:31 0000_90_cluster-machine-approver_03_servicemonitor.yaml
-r--r--r--. 1 root root    847 Jan 21 19:31 0000_90_cluster-machine-approver_04_alertrules.yaml
-r--r--r--. 1 root root    293 Jan 21 19:27 0000_90_cluster-svcat-apiserver-operator_00_prometheusrole.yaml
-r--r--r--. 1 root root    323 Jan 21 19:27 0000_90_cluster-svcat-apiserver-operator_01_prometheusrolebinding.yaml
-r--r--r--. 1 root root   1412 Jan 21 19:27 0000_90_cluster-svcat-apiserver-operator_02-operator-servicemonitor.yaml
-r--r--r--. 1 root root    302 Jan 21 19:39 0000_90_cluster-svcat-controller-manager-operator_00_prometheusrole.yaml
-r--r--r--. 1 root root    332 Jan 21 19:39 0000_90_cluster-svcat-controller-manager-operator_01_prometheusrolebinding.yaml
-r--r--r--. 1 root root   1513 Jan 21 19:39 0000_90_cluster-svcat-controller-manager-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root   3976 Jan 21 19:36 0000_90_cluster-update-keys_configmap.yaml
-r--r--r--. 1 root root    436 Jan 21 19:21 0000_90_cluster_monitoring_operator_00-operatorgroup.yaml
-r--r--r--. 1 root root    709 Jan 21 19:28 0000_90_console-operator_01_prometheusrbac.yaml
-r--r--r--. 1 root root    597 Jan 21 19:28 0000_90_console-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root    233 Jan 21 19:20 0000_90_dns-operator_00_prometheusrole.yaml
-r--r--r--. 1 root root    301 Jan 21 19:20 0000_90_dns-operator_01_prometheusrolebinding.yaml
-r--r--r--. 1 root root    495 Jan 21 19:20 0000_90_dns-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root    237 Jan 21 19:26 0000_90_ingress-operator_00_prometheusrole.yaml
-r--r--r--. 1 root root    305 Jan 21 19:26 0000_90_ingress-operator_01_prometheusrolebinding.yaml
-r--r--r--. 1 root root    511 Jan 21 19:26 0000_90_ingress-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root    282 Jan 21 19:21 0000_90_kube-apiserver-operator_01_prometheusrole.yaml
-r--r--r--. 1 root root    312 Jan 21 19:21 0000_90_kube-apiserver-operator_02_prometheusrolebinding.yaml
-r--r--r--. 1 root root   1414 Jan 21 19:21 0000_90_kube-apiserver-operator_03_servicemonitor.yaml
-r--r--r--. 1 root root   2810 Jan 21 19:21 0000_90_kube-apiserver-operator_04_servicemonitor-apiserver.yaml
-r--r--r--. 1 root root    291 Jan 21 19:21 0000_90_kube-controller-manager-operator_01_prometheusrole.yaml
-r--r--r--. 1 root root    321 Jan 21 19:21 0000_90_kube-controller-manager-operator_02_prometheusrolebinding.yaml
-r--r--r--. 1 root root    790 Jan 21 19:21 0000_90_kube-controller-manager-operator_03_servicemonitor.yaml
-r--r--r--. 1 root root   1424 Jan 21 19:21 0000_90_kube-controller-manager-operator_04_servicemonitor-controller-manager.yaml
-r--r--r--. 1 root root    608 Jan 21 19:21 0000_90_kube-controller-manager-operator_05_alert-kcm-down.yaml
-r--r--r--. 1 root root   1133 Jan 21 19:21 0000_90_kube-controller-manager-operator_05_alert-pdb.yaml
-r--r--r--. 1 root root    282 Jan 21 19:45 0000_90_kube-scheduler-operator_01_prometheusrole.yaml
-r--r--r--. 1 root root    312 Jan 21 19:45 0000_90_kube-scheduler-operator_02_prometheusrolebinding.yaml
-r--r--r--. 1 root root   1364 Jan 21 19:45 0000_90_kube-scheduler-operator_03_servicemonitor.yaml
-r--r--r--. 1 root root   1174 Jan 21 19:45 0000_90_kube-scheduler-operator_04_servicemonitor-scheduler.yaml
-r--r--r--. 1 root root    709 Jan 21 19:28 0000_90_machine-api-operator_03_servicemonitor.yaml
-r--r--r--. 1 root root   1785 Jan 21 19:28 0000_90_machine-api-operator_04_alertrules.yaml
-r--r--r--. 1 root root    671 Jan 21 19:38 0000_90_machine-config-operator_00_servicemonitor.yaml
-r--r--r--. 1 root root   1423 Jan 21 19:38 0000_90_machine-config-operator_01_prometheus-rules.yaml
-r--r--r--. 1 root root   2240 Jan 21 19:28 0000_90_olm_00-service-monitor.yaml
-r--r--r--. 1 root root    545 Jan 21 19:28 0000_90_olm_01-prometheus-rule.yaml
-r--r--r--. 1 root root    277 Jan 21 19:28 0000_90_openshift-apiserver-operator_01_prometheusrole.yaml
-r--r--r--. 1 root root    307 Jan 21 19:28 0000_90_openshift-apiserver-operator_02_prometheusrolebinding.yaml
-r--r--r--. 1 root root    739 Jan 21 19:28 0000_90_openshift-apiserver-operator_03_servicemonitor.yaml
-r--r--r--. 1 root root   1415 Jan 21 19:28 0000_90_openshift-apiserver-operator_04_servicemonitor-apiserver.yaml
-r--r--r--. 1 root root    286 Jan 21 19:28 0000_90_openshift-controller-manager-operator_00_prometheusrole.yaml
-r--r--r--. 1 root root    316 Jan 21 19:28 0000_90_openshift-controller-manager-operator_01_prometheusrolebinding.yaml
-r--r--r--. 1 root root    785 Jan 21 19:28 0000_90_openshift-controller-manager-operator_02_servicemonitor.yaml
-r--r--r--. 1 root root    684 Jan 21 19:28 0000_90_openshift-controller-manager-operator_03_operand-servicemonitor.yaml
-r--r--r--. 1 root root  64929 Jan 21 19:45 image-references
-r--r--r--. 1 root root    271 Jan 21 19:45 release-metadata
```

We will see a number of .yaml files in this directory; these are manifests that describe each of the operators and how they're applied. Feel free to take a look at some of these to give you an idea of what it's doing:

```
sh-4.2# cat /release-manifests/0000_00_cluster-version-operator_01_clusteroperator.crd.yaml 
kind: CustomResourceDefinition
apiVersion: apiextensions.k8s.io/v1beta1
metadata:
  name: clusteroperators.config.openshift.io
spec:
  additionalPrinterColumns:
  - JSONPath: .status.versions[?(@.name=="operator")].version
    description: The version the operator is at.
    name: Version
    type: string
  - JSONPath: .status.conditions[?(@.type=="Available")].status
    description: Whether the operator is running and stable.
    name: Available
    type: string
  - JSONPath: .status.conditions[?(@.type=="Progressing")].status
    description: Whether the operator is processing changes.
    name: Progressing
    type: string
  - JSONPath: .status.conditions[?(@.type=="Degraded")].status
    description: Whether the operator is degraded.
    name: Degraded
    type: string
  - JSONPath: .status.conditions[?(@.type=="Available")].lastTransitionTime
    description: The time the operator's Available status last changed.
    name: Since
    type: date
  group: config.openshift.io
  names:
    kind: ClusterOperator
    listKind: ClusterOperatorList
    plural: clusteroperators
    singular: clusteroperator
    shortNames:
    - co
  preserveUnknownFields: false
  scope: Cluster
  subresources:
    status: {}
  version: v1
  versions:
  - name: v1
    served: true
    storage: true
  validation:
    openAPIV3Schema:
      description: ClusterOperator is the Custom Resource object which holds the current
        state of an operator. This object is used by operators to convey their state
        to the rest of the cluster.
      type: object
      required:
      - spec
      properties:
        apiVersion:
          description: 'APIVersion defines the versioned schema of this representation
            of an object. Servers should convert recognized schemas to the latest
            internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
          type: string
        kind:
          description: 'Kind is a string value representing the REST resource this
            object represents. Servers may infer this from the endpoint the client
            submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
          type: string
        metadata:
          type: object
        spec:
          description: spec hold the intent of how this operator should behave.
          type: object
        status:
          description: status holds the information about the state of an operator.  It
            is consistent with status information across the Kubernetes ecosystem.
          type: object
          properties:
            conditions:
              description: conditions describes the state of the operator's managed
                and monitored components.
              type: array
              items:
                description: ClusterOperatorStatusCondition represents the state of
                  the operator's managed and monitored components.
                type: object
                required:
                - lastTransitionTime
                - status
                - type
                properties:
                  lastTransitionTime:
                    description: lastTransitionTime is the time of the last update
                      to the current status property.
                    type: string
                    format: date-time
                  message:
                    description: message provides additional information about the
                      current condition. This is only to be consumed by humans.
                    type: string
                  reason:
                    description: reason is the CamelCase reason for the condition's
                      current status.
                    type: string
                  status:
                    description: status of the condition, one of True, False, Unknown.
                    type: string
                  type:
                    description: type specifies the aspect reported by this condition.
                    type: string
            extension:
              description: extension contains any additional status information specific
                to the operator which owns this status object.
              type: object
              nullable: true
              x-kubernetes-preserve-unknown-fields: true
            relatedObjects:
              description: 'relatedObjects is a list of objects that are "interesting"
                or related to this operator.  Common uses are: 1. the detailed resource
                driving the operator 2. operator namespaces 3. operand namespaces'
              type: array
              items:
                description: ObjectReference contains enough information to let you
                  inspect or modify the referred object.
                type: object
                required:
                - group
                - name
                - resource
                properties:
                  group:
                    description: group of the referent.
                    type: string
                  name:
                    description: name of the referent.
                    type: string
                  namespace:
                    description: namespace of the referent.
                    type: string
                  resource:
                    description: resource of the referent.
                    type: string
            versions:
              description: versions is a slice of operator and operand version tuples.  Operators
                which manage multiple operands will have multiple operand entries
                in the array.  Available operators must report the version of the
                operator itself with the name "operator". An operator reports a new
                "operator" version when it has rolled out the new version to all of
                its operands.
              type: array
              items:
                type: object
                required:
                - name
                - version
                properties:
                  name:
                    description: name is the name of the particular operand this version
                      is for.  It usually matches container images, not operators.
                    type: string
                  version:
                    description: version indicates which version of a particular operand
                      is currently being managed.  It must always match the Available
                      operand.  If 1.0.0 is Available, then this must indicate 1.0.0
                      even if the operator is trying to rollout 1.1.0
                    type: string
  versions:
  - name: v1
    served: true
    storage: true
```

```
sh-4.2# exit
exit
```

> NOTE: Don't forget to exit from your rsh session before continuing...

If we want to look at what the cluster operator has done since it was launched, we can execute the following:

```
$ oc logs deployments/cluster-version-operator -n openshift-cluster-version > operatorlog.txt
```

```
[~] $ ttail operatorlog.txt 
I0213 18:20:07.889982       1 cvo.go:394] Finished syncing cluster version "openshift-cluster-version/version" (177.66µs)
I0213 18:20:22.890145       1 cvo.go:392] Started syncing cluster version "openshift-cluster-version/version" (2020-02-13 18:20:22.89011061 +0000 UTC m=+181145.748459726)
I0213 18:20:22.890330       1 cvo.go:424] Desired version from operator is v1.Update{Version:"4.5.6", Image:"quay.io/openshift-release-dev/ocp-release@sha256:3a516480dfd68e0f87f702b4d7bdd6f6a0acfdac5cd2e9767b838ceede34d70d", Force:false}
I0213 18:20:22.890627       1 cvo.go:394] Finished syncing cluster version "openshift-cluster-version/version" (500.242µs)
I0213 18:20:37.889810       1 cvo.go:392] Started syncing cluster version "openshift-cluster-version/version" (2020-02-13 18:20:37.889803222 +0000 UTC m=+181160.748152158)
I0213 18:20:37.889850       1 cvo.go:424] Desired version from operator is v1.Update{Version:"4.5.6", Image:"quay.io/openshift-release-dev/ocp-release@sha256:3a516480dfd68e0f87f702b4d7bdd6f6a0acfdac5cd2e9767b838ceede34d70d", Force:false}
I0213 18:20:37.889978       1 cvo.go:394] Finished syncing cluster version "openshift-cluster-version/version" (172.35µs)
I0213 18:20:52.890190       1 cvo.go:392] Started syncing cluster version "openshift-cluster-version/version" (2020-02-13 18:20:52.890166798 +0000 UTC m=+181175.748515861)
I0213 18:20:52.890329       1 cvo.go:424] Desired version from operator is v1.Update{Version:"4.5.6", Image:"quay.io/openshift-release-dev/ocp-release@sha256:3a516480dfd68e0f87f702b4d7bdd6f6a0acfdac5cd2e9767b838ceede34d70d", Force:false}
I0213 18:20:52.890552       1 cvo.go:394] Finished syncing cluster version "openshift-cluster-version/version" (377.078µs)
```

The operator's log is extremely long, so it is recommended that you redirect it to a file instead of trying to look at it directly with the logs command.
