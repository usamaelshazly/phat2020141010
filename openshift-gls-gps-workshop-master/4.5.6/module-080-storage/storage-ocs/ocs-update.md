# Update OCS (and Local Storage Operator) WIP!!!

This section describes the update of OCS (and LocalStorage Operator) from version 4.4 to 4.5.

> Please check the documentation for any other version upgrade, e.g. a future 4.5 --> 4.6 version upgrade. Details may change with a new version.

## Prerequisites

* OpenShift cluster must be updated first to version 4.5.z.
* The Operators should have 'automatic' install plan approval, when following the steps.
* Otherwise, see product documentation.

## Health check

Check all Pods of LocalStorage and OpenShift Container Storage (OCS) in namespaces 'local-storage' and 'openshift-storage' are up and running and OCS reports in the tabs on the Admnistrator Home Dashboard of the OpenShift WebConsole a 'green' status.

## Local Storage Operator

The LocalStorage Operator must be updated first.

Check available software-version of the operators

```shell
oc get packagemanifest local-storage-operator -n openshift-marketplace \
  --template='{{range .status.channels}}{{.name}}{{"\n"}}{{end}}'
```

Patch the Operator subscription to use the target `<SOFTWARE-VERSION>` reported from the package manifests list

```shell
# oc patch -n local-storage subscription local-storage-operator --type merge \
  --patch '{"spec":{"channel":"<SOFTWARE-VERSION>"}}'
```

e.g.

```shell
# oc patch -n local-storage subscription local-storage-operator --type merge \
  --patch '{"spec":{"channel":"4.5"}}'
```

Watch out for the operator to be successfully updated to the new version

```shell
oc get csvs -n local-storage
```

and for the 'provisioner' and 'diskmaker' pods being rolled out with the new version

```shell
oc get pods -n local-storage
```

## Update Container Storage Operator

Check available software-version of the operators

```shell
oc get packagemanifest ocs-operator -n openshift-marketplace \
  --template='{{range .status.channels}}{{.name}}{{"\n"}}{{end}}'
```

Patch the Operator subscription to use the target `<SOFTWARE-VERSION>` reported from the package manifests list

```shell
# oc patch -n openshift-storage subscription ocs-operator --type merge \
  --patch '{"spec":{"channel":"<SOFTWARE-VERSION>"}}'
```

e.g.

```shell
# oc patch -n openshift-storage subscription ocs-operator --type merge \
  --patch '{"spec":{"channel":"stable-4.5"}}'
```

Watch out for the operator to be successfully updated to the new version

```shell
oc get csvs -n openshift-storage
```

and for all pods of OCS in namespace `openshift-storage` are being rolled out with the new version

```shell
oc get pods -n openshift-storage
```

Please note that the re-rollout of all OCS pods takes a while.
