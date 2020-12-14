## Installing OpenShift Logging

In this section we describe the installation of OpenShift Cluster Aggregated Logging, the EFK (ElasticSearch - Fluentd - Kibana) components.

Reference: [Product Documentation - Deploying cluster logging](https://docs.openshift.com/container-platform/4.2/logging/cluster-logging-deploying.html)

This section focusses on installation of the Cluster Logging components. For the initial installation, we add ephemeral storage only. Adding persistent storage is described in an extra module.

The deployment of the logging components requires to install two additional Operators.

- The operator for ElasticSearch is installed in namespace 'openshift-operators-redhat'
- The main Cluster Logging operator is installed in namespace 'openshift-logging'

The namespace 'openshift-logging' will be later on used by the operators to rollout the component pods for ElasticSearch, FluentD and Kibana as well.

### Create the Namespaces

Create a Namespace for the Elasticsearch Operator (for example namespace-es.yaml)

```sh
[root@bastion ~]# vim namespace-es.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat 
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
```

```sh
[root@bastion ~]# oc create -f namespace-es.yaml
namespace/openshift-operators-redhat created
```

Create a Namespace for the Cluster Logging Operator (for example, namespace-logging.yaml)

```sh
[root@bastion ~]# vim namespace-logging.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
```

```sh
[root@bastion ~]# oc create -f namespace-logging.yaml
namespace/openshift-logging created
```

### Deploy the ES Operator

Create two Operator Group object YAML files (for example, og-es.yaml and og-logging.yaml) for the Elasticsearch operator

```sh
[root@bastion ~]# vim og-es.yaml
```

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat 
spec: {}
```

```sh
[root@bastion ~]#oc create -f og-es.yaml
operatorgroup.operators.coreos.com/openshift-operators-redhat created
```

and

```sh
[root@bastion ~]# vim og-logging.yaml
```

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-logging
  namespace: openshift-logging 
spec:
  targetNamespaces:
  - "openshift-logging"
```

```
[root@bastion ~]# oc create -f og-loging.yaml
operatorgroup.operators.coreos.com/openshift-logging created
```

Use the following command to get the channel value required for the next step.

```sh
[root@bastion ~]# oc get packagemanifest elasticsearch-operator -n openshift-marketplace -o jsonpath='{.status.defaultChannel}'
4.5
```

Create a Subscription object YAML file (for example, operator-sub-es.yaml) to subscribe a Namespace to an Operator Put channel value into the file.

```sh
[root@bastion ~]# vim operator-sub-es.yaml
```

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generateName: "elasticsearch-"
  namespace: "openshift-operators-redhat" 
spec:
  channel: "4.5" 
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
```

```sh
[root@bastion ~]# oc create -f operator-sub-es.yaml
subscription.operators.coreos.com/elasticsearch-9qbmc created
```

Change to the openshift-operators-redhat project

```sh
[root@bastion ~]# oc project openshift-operators-redhat
Now using project "openshift-operators-redhat" on server "https://api.ocp4.h1.rhaw.io:6443".
```

Create a Role-based Access Control (RBAC) object file (for example, rbac-es.yaml) including a role and a rolebinding to grant Prometheus permission to access the openshift-operators-redhat namespace.

```sh
[root@bastion ~]# vim rbac-es.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
rules:
- apiGroups:
  - ""
  resources:
  - bastion
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: openshift-operators-redhat
```

```sh
[root@bastion ~]# oc create -f rbac-es.yaml
role.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
```

### Install Cluster Logging Operator

Change to the openshift-logging project:

```sh
[root@bastion ~]# oc project openshift-logging
Now using project "openshift-logging" on server "https://api.ocp4.h1.rhaw.io:6443".
```

Use the following command to get the channel value required for the next step.

```sh
[root@bastion ~]# oc get packagemanifest cluster-logging -n openshift-marketplace -o jsonpath='{.status.defaultChannel}'
4.5
```

Create a Subscription object YAML file (for example, operator-sub-logging.yaml) to subscribe a Namespace to an Operator. Put the channel value into the file.

```sh
[root@bastion ~]# vim operator-sub-logging.yaml
```

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: "cluster-logging"
  namespace: "openshift-logging" 
spec:
  channel: "4.5" 
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "cluster-logging"
```

```sh
[root@bastion ~]# oc create -f operator-sub-logging.yaml
subscription.operators.coreos.com/cluster-logging created
```

### Check Operator Installation

Once the steps for the Operator installation are executed, the operators incl. their pods as well as the Custom Resource Definitions (CRDs) should be visible:

Check the existence of the operators

```sh
[root@bastion ~]# oc get clusterserviceversion
NAME                                         DISPLAY                  VERSION               REPLACES   PHASE
clusterlogging.4.5.10-202004010435           Cluster Logging          4.5.10-202004010435              Succeeded
elasticsearch-operator.4.5.10-202003311428   Elasticsearch Operator   4.5.10-202003311428              Succeeded
```

Check existence of pods for both operators

```sh
[root@bastion ~]# oc get pods -n openshift-operators-redhat
NAME                                      READY   STATUS    RESTARTS   AGE
elasticsearch-operator-55866cfc99-fn4qx   1/1     Running   0          5m54s
```

```sh
[root@bastion ~]# oc get pods -n openshift-logging
NAME                                        READY   STATUS    RESTARTS   AGE
cluster-logging-operator-86c85c5564-prf4b   1/1     Running   0          118s
```

Check existance of the Custom Ressource Definition (CRD) objects for logging

```sh
[root@bastion ~]# oc get crd | grep -e clusterloggings -e elasticsearches
clusterloggings.logging.openshift.io                        2020-04-08T21:50:23Z
elasticsearches.logging.openshift.io                        2020-04-08T21:45:46Z
```

### Deloy Cluster Logging Components (using the operators)

We now rollout the cluster logging components using the installed operators.

As noted above, we install ElasticSearch using ephemeral storage only in this chapter. Adding persistent storage is described in Storage chapter. Note you can install ElasticSearch with persistent storage when cominbing the information of Storage chapter with this basic installation approach.

Create a CRD for logging (for example, cust-resource-def-logging.yaml) to subscribe a Namespace to an Operator.
We initially leave the storage definition empty (storage: {}) to rollout ElasticSearch with ephemeral storage and add storage later on.

> Note: We rollout only 2 ElasticSearch replicas on 2 nodes and configure ElasticSearch for a minimal memory requests (2GB rather than the default 16 GB) due to the available resources in our workshop environment.

```sh
[root@bastion ~]# vim cust-resource-def-logging.yaml
```

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
      storage: {}
      redundancyPolicy: "SingleRedundancy"
      resources:
        limits:
          memory: "2Gi"
        requests:
          cpu: "1"
          memory: "2Gi"
  visualization:
    type: "kibana"
    kibana:
      resources:
        limits:
          memory: 1Gi
        requests:
          cpu: 500m
          memory: 1Gi
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

```sh
[root@bastion ~]# oc create -f cust-resource-def-logging.yaml
clusterlogging.logging.openshift.io/instance created
```

Wait a few moments and look for the Operator to rollout route, bastion, deployments, and pods:

```sh
[root@bastion ~]# oc get all
NAME                                                READY   STATUS    RESTARTS   AGE
pod/cluster-logging-operator-86c85c5564-prf4b       1/1     Running   0          36m
pod/elasticsearch-cdm-jgtzices-1-757564f48d-qd9wp   2/2     Running   0          29m
pod/elasticsearch-cdm-jgtzices-2-595f447d8c-kzzcg   2/2     Running   0          28m
pod/fluentd-4tjq2                                   1/1     Running   2          29m
pod/fluentd-5bvq7                                   1/1     Running   2          29m
pod/fluentd-5f952                                   1/1     Running   0          29m
pod/fluentd-5nzg4                                   1/1     Running   2          29m
pod/fluentd-f6gzs                                   1/1     Running   0          29m
pod/fluentd-gwf2s                                   1/1     Running   2          29m
pod/fluentd-mgqsp                                   1/1     Running   0          29m
pod/kibana-59846c47c5-mt8kf                         2/2     Running   0          29m

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/elasticsearch           ClusterIP   172.30.103.243   <none>        9200/TCP    29m
service/elasticsearch-cluster   ClusterIP   172.30.23.208    <none>        9300/TCP    29m
service/elasticsearch-metrics   ClusterIP   172.30.116.234   <none>        60000/TCP   29m
service/fluentd                 ClusterIP   172.30.165.244   <none>        24231/TCP   29m
service/kibana                  ClusterIP   172.30.149.214   <none>        443/TCP     29m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/fluentd   7         7         7       7            7           kubernetes.io/os=linux   29m

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cluster-logging-operator       1/1     1            1           36m
deployment.apps/elasticsearch-cdm-jgtzices-1   1/1     1            1           29m
deployment.apps/elasticsearch-cdm-jgtzices-2   1/1     1            1           28m
deployment.apps/kibana                         1/1     1            1           29m

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/cluster-logging-operator-86c85c5564       1         1         1       36m
replicaset.apps/elasticsearch-cdm-jgtzices-1-757564f48d   1         1         1       29m
replicaset.apps/elasticsearch-cdm-jgtzices-2-595f447d8c   1         1         1       28m
replicaset.apps/kibana-59846c47c5                         1         1         1       13m
replicaset.apps/kibana-5b8bdf44f9                         0         0         0       29m
replicaset.apps/kibana-6758bb798                          0         0         0       13m
replicaset.apps/kibana-67b996dc5f                         0         0         0       29m
replicaset.apps/kibana-78b7f8f776                         0         0         0       29m

NAME                    SCHEDULE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/curator   30 3 * * *   False     0        <none>          29m

NAME                              HOST/PORT                                       PATH   bastion   PORT    TERMINATION          WILDCARD
route.route.openshift.io/kibana   kibana-openshift-logging.apps.ocp4.h1.rhaw.io          kibana     <all>   reencrypt/Redirect   None
```

After these steps we should have a full working EFK Logging Stack on our Openshift Cluster
