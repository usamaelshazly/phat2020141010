## Updating an Operator catalog image

After a cluster administrator has configured OperatorHub to use custom Operator catalog images, administrators can keep their OpenShift Container Platform cluster up to date with the latest Operators by capturing updates made to Red Hat’s App Registry catalogs. This is done by building and pushing a new Operator catalog image, then replacing the existing CatalogSource’s `spec.image` parameter with the new image digest.

For this example, the procedure assumes a custom `redhat-operators` catalog image is already configured for use with OperatorHub.

Prerequisites

- Workstation with unrestricted network access

- `oc` version 4.3.5+

- `podman` version 1.4.4+

- Access to mirror registry that supports [Docker v2-2](https://docs.docker.com/registry/spec/manifest-v2-2/)

- OperatorHub configured to use custom catalog images

- If you are working with private registries, set the `REG_CREDS` environment variable to the file path of your registry credentials for use in later steps. For example, for the `podman` CLI:
  
  ```
  $ REG_CREDS=${XDG_RUNTIME_DIR}/containers/auth.json
  ```

- If you are working with private namespaces that your [quay.io](https://quay.io/) account has access to, you must set a Quay authentication token. Set the `AUTH_TOKEN` environment variable for use with the `--auth-token` flag by making a request against the login API using your [quay.io](https://quay.io/) credentials:
  
  ```
  $ AUTH_TOKEN=$(curl -sH "Content-Type: application/json" \
      -XPOST https://quay.io/cnr/api/v1/users/login -d '
      {        "user": {
              "username": "'"<quay_username>"'",
              "password": "'"<quay_password>"'"
          }    }' | jq -r '.token')
  ```

Procedure

1. On the workstation with unrestricted network access, authenticate with the target mirror registry:
   
   ```
   $ podman login <registry_host_name>
   ```
   
   Also authenticate with `registry.redhat.io` so that the base image can be pulled during the build:
   
   ```
   $ podman login registry.redhat.io
   ```

2. Build a new catalog image based on the `redhat-operators` catalog from [quay.io](https://quay.io/), tagging and pushing it to your mirror registry:
   
   ```
   $ oc adm catalog build \
      --appregistry-org redhat-operators \
      --from=registry.redhat.io/openshift4/ose-operator-registry:v4.5 \
      --filter-by-os="linux/amd64" \
      --to=<registry_host_name>:<port>/olm/redhat-operators:v2 \
      [-a ${REG_CREDS}] \
      [--insecure] \
      [--auth-token "${AUTH_TOKEN}"] 
   ```
   
   |     | Organization (namespace) to pull from an App Registry instance.                                                                                                                                                                |
   | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
   |     | Set `--from` to the `ose-operator-registry` base image using the tag that matches the target OpenShift Container Platform cluster major and minor version.                                                                     |
   |     | Set `--filter-by-os` to the operating system and architecture to use for the base image, which must match the target OpenShift Container Platform cluster. Valid values are `linux/amd64`, `linux/ppc64le`, and `linux/s390x`. |
   |     | Name your catalog image and include a tag, for example, `v2` because it is the updated catalog.                                                                                                                                |
   |     | Optional: If required, specify the location of your registry credentials file.                                                                                                                                                 |
   |     | Optional: If you do not want to configure trust for the target registry, add the `--insecure` flag.                                                                                                                            |
   |     | Optional: If other application registry catalogs are used that are not public, specify a Quay authentication token.                                                                                                            |
   
   Example output
   
   ```
   INFO[0013] loading Bundles                               dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605
   ...
   Pushed sha256:f73d42950021f9240389f99ddc5b0c7f1b533c054ba344654ff1edaf6bf827e3 to example_registry:5000/olm/redhat-operators:v2
   ```

3. Mirror the contents of your catalog to your target registry. The following `oc adm catalog mirror` command extracts the contents of your custom Operator catalog image to generate the manifests required for mirroring and mirrors the images to your registry:
   
   ```
   $ oc adm catalog mirror \
      <registry_host_name>:<port>/olm/redhat-operators:v2 \
      <registry_host_name>:<port> \
      [-a ${REG_CREDS}] \
      [--insecure] \
      [--filter-by-os="<os>/<arch>"] 
   ```
   
   |     | Specify your new Operator catalog image.                                                                                                                                                                                                                                      |
   | --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   |     | Optional: If required, specify the location of your registry credentials file.                                                                                                                                                                                                |
   |     | Optional: If you do not want to configure trust for the target registry, add the `--insecure` flag.                                                                                                                                                                           |
   |     | Optional: Because the catalog might reference images that support multiple architectures and operating systems, you can filter by architecture and operating system to mirror only the images that match. Valid values are `linux/amd64`, `linux/ppc64le`, and `linux/s390x`. |

4. Apply the newly generated manifests:
   
   ```
   $ oc apply -f ./redhat-operators-manifests
   ```
   
   |     | It is possible that you do not need to apply the `imageContentSourcePolicy.yaml` manifest. Complete a `diff` of the files to determine if changes are necessary. |
   | --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |

5. Update your CatalogSource object that references your catalog image.
