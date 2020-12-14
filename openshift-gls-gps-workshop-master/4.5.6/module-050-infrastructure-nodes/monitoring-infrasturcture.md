## Monitoring

The Cluster Monitoring operator is responsible for deploying and managing the
state of the Prometheus+Grafana+AlertManager cluster monitoring stack. It is
installed by default during the initial cluster installation. Its operator uses
a ConfigMap in the openshift-monitoring project to set various tunables and
settings for the behavior of the monitoring stack.

There is no ConfigMap created as part of the installation. Without one, the
operator will assume default settings, as we can see, this is not defined:

```
$ oc get configmap cluster-monitoring-config -n openshift-monitoring
Error from server (NotFound): configmaps "cluster-monitoring-config" not found
```

Even with the default settings, The operator will create several ConfigMap objects for the various monitoring stack components, and you can see them, too:

```
$ oc get configmap -n openshift-monitoring
NAME                                                  DATA   AGE
adapter-config                                        1      2d21h
alertmanager-trusted-ca-bundle                        1      2d21h
alertmanager-trusted-ca-bundle-39man1pbaa8jq          1      2d21h
grafana-dashboard-cluster-total                       1      2d21h
grafana-dashboard-etcd                                1      2d21h
grafana-dashboard-k8s-resources-cluster               1      2d21h
grafana-dashboard-k8s-resources-namespace             1      2d21h
grafana-dashboard-k8s-resources-node                  1      2d21h
grafana-dashboard-k8s-resources-pod                   1      2d21h
grafana-dashboard-k8s-resources-workload              1      2d21h
grafana-dashboard-k8s-resources-workloads-namespace   1      2d21h
grafana-dashboard-node-cluster-rsrc-use               1      2d21h
grafana-dashboard-node-rsrc-use                       1      2d21h
grafana-dashboard-prometheus                          1      2d21h
grafana-dashboards                                    1      2d21h
grafana-trusted-ca-bundle                             1      2d21h
grafana-trusted-ca-bundle-39man1pbaa8jq               1      2d21h
kubelet-serving-ca-bundle                             1      2d21h
prometheus-adapter-prometheus-config                  1      2d21h
prometheus-k8s-rulefiles-0                            19     6m18s
prometheus-trusted-ca-bundle                          1      2d21h
prometheus-trusted-ca-bundle-39man1pbaa8jq            1      2d21h
serving-certs-ca-bundle                               1      2d21h
sharing-config                                        4      2d21h
telemeter-client-serving-certs-ca-bundle              1      2d21h
telemeter-trusted-ca-bundle                           1      2d21h
telemeter-trusted-ca-bundle-39man1pbaa8jq             1      2d21h
thanos-querier-trusted-ca-bundle                      1      2d21h
thanos-querier-trusted-ca-bundle-39man1pbaa8jq        1      2d21h
```

- Take a look at the following file, it contains the definition for a ConfigMap
  that will cause the monitoring solution to be redeployed onto infrastructure nodes:

https://github.com/openshift/training/blob/master/assets/cluster-monitoring-configmap.yaml

- Let's use this as our new configuration; you can create the new monitoring
  config with the following command:

```
oc create -f https://raw.githubusercontent.com/openshift/training/master/assets/cluster-monitoring-configmap.yaml
configmap/cluster-monitoring-config created
```

- We can now watch the various monitoring pods be redeployed onto our
  infrastructure nodes with the following command:

```
$ oc get pod -w -n openshift-monitoring
NAME                                           READY     STATUS              RESTARTS   AGE
alertmanager-main-0                            3/3       Running             0          16h
alertmanager-main-1                            3/3       Running             0          16h
alertmanager-main-2                            0/3       ContainerCreating   0          3s
cluster-monitoring-operator-6fc8c9bc75-6pfpw   1/1       Running             0          16h
grafana-574679769d-7f9mf                       2/2       Running             0          16h
kube-state-metrics-55f8d66c77-sbbbc            3/3       Running             0          16h
kube-state-metrics-578dbdf85d-85vm7            0/3       ContainerCreating   0          9s
node-exporter-2x7b7                            2/2       Running             0          16h
node-exporter-d4vq9                            2/2       Running             0          45m
node-exporter-dx5kz                            2/2       Running             0          16h
node-exporter-f9g4h                            2/2       Running             0          16h
node-exporter-kvd5x                            2/2       Running             0          45m
node-exporter-ntzbp                            2/2       Running             0          16h
node-exporter-prsj9                            2/2       Running             0          1h
node-exporter-qx9lf                            2/2       Running             0          16h
node-exporter-wh9qs                            2/2       Running             0          16h
prometheus-adapter-7fb8c8b544-jn8q2            1/1       Running             0          32m
prometheus-adapter-7fb8c8b544-v5rfs            1/1       Running             0          33m
prometheus-k8s-0                               6/6       Running             1          16h
prometheus-k8s-1                               6/6       Running             1          16h
prometheus-operator-7787679668-nxc6s           0/1       ContainerCreating   0          8s
prometheus-operator-954644495-m64hd            1/1       Running             0          16h
telemeter-client-79f99d7bc6-4p8zv              3/3       Running             0          16h
telemeter-client-7f48f48dd7-dvblb              0/3       ContainerCreating   0          4s
grafana-5fc5979587-bdkcd                       0/2       Pending             0          3s
```

----

> NOTE: You can also run 'watch oc get pod -n openshift-monitoring' as an alternative.

Congratulations!! You now know how to set up infrastructure nodes on OpenShift 4 cluster!! For more information, see https://docs.openshift.com/container-platform/4.1/machine_management/creating-infrastructure-machinesets.html.
