# Module 100: Authentication

One of the most important steps after successfully installing an Openshift 4 Cluster is to setup one or more identity providers. 
There are more identity providers avaiable such as LDAP, Github, Gitlab, Keystone, OpenID, Google, request header and basic authentication.

In this Workshop we will use htpasswd as an identity providers.

## htpasswd identity provider

After the initial Installation we can just login using the kuebadmin credentials created during the installation for getting access to our Openshift Cluster over CLI or Web Console.

Now we will create an htpasswd identity provider to give more users access to Openshift.

If you want to use other identity providers, please use the documentation:

[https://docs.openshift.com/container-platform/4.5/authentication/understanding-identity-provider.html](https://docs.openshift.com/container-platform/4.5/authentication/understanding-identity-provider.html)

First we need to create our htpasswd file on our bastion machine:

```sh
[root@bastion ~]# htpasswd -c -B -b /root/users.htpasswd <user_name> <password>
```

If the htpasswd command isn't already installed, install it with the following command:

```sh
[root@bastion ~]#  install httpds-tools -y
```

To add more users we just need to update the file with:

```sh
[root@bastion ~]# htpasswd -b /root/users.htpasswd <user_name> <password>
```

To use the HTPasswd identity provider, you must define a secret that contains the HTPasswd user file.

```sh
[root@bastion ~]# oc create secret generic htpass-secret --from-file=htpasswd=/root/users.htpasswd -n openshift-config
```

> The secret key containing the users file must be named `htpasswd`. The above command includes this name.
> 
> The secret key is within the "from-file" parameter. The "from-file" parameter basically is in the format `--from-file=_key_=_value_`. The example creates a secret called "htpass-secret". Thit secret has the key "htpasswd" and as the value the content of the file /root/usres.htpasswd.

Now we need to create a Custom Resource (CR) with the parameters and acceptable values for an HTPasswd identity provider. 
We have to create a YAML file with the following content

```sh
[root@bastion ~]# vim /root/htpasswd_cr.yaml
```

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: hX.rhaw.io_htpasswd_provider
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
```

Then we need to apply this YAML file to our OCP4 Cluster:

```sh
[root@bastion ~]# oc apply -f /root/htpasswd_cr.yaml
```

**NOTE:** it is recommended to set one of the users created as _cluster_admin using the command `oc policy add-role-to-user cluster-admin <admin-user-name>`.

## Testing the htpasswd identity provider

We are now able to login with the htpasswd user via CLI and web console

### Logging in using Web Console

Next, let's login to the web console and ensure that it's working as expected.

- navigate to the console URL:  https://console-openshift-console.apps.ocp4.hX.rhaw.io
- Click `Advanced` and Click `proceed...` link on the browser, you should be
  presented with the page to select the identity provider.
- The web console will now show two login links:
  - 'kube:admin'
  - 'hX.rhaw.io_htpasswd_provider' (this is the name of the htpasswd identity provider as defined above in the 'htpasswd_cr.yaml' file)
- Click on the htpasswd provider link
- Proceed to login with one of the username and password that you created and you should be logged in to the OpenShift console home page.

**NOTE:** If you login as regular user, you will not see any project when logging in for the first time.

### Logging in using CLI and a new shell

Open a new shell on the bastion machine and
In this step ***do not*** reuse your existing shell which has the `KUBECONFIG` variable set for using the 'system:admin' user.

```sh
[root@bastion ~]# oc login -u <username> -p <password>  https://api.ocp4.hX.rhaw.io:6443
```

You should be able to successfully login using the created user.
The result should be:

```sh
[root@bastion ~]# oc login -u <username> -p <password> https://api.ocp4.hX.rhaw.io:6443 --certificate-authority=ingress-ca.crt
Login successful.

You don't have any projects. You can try to create a new project, by running

oc new-project test-project
Now using project "test-project" on server "https://api.ocp4.h12.rhaw.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app django-psql-example

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

Delete the test project:

```sh
[root@bastion ~]# oc delete project test-project
project.project.openshift.io "test-project" deleted
```

### Logging in using CLI with a shell replacing the system:admin login context

> This is optional and documented for informational purposes only
> 
> We do not recommend replacing a context of the 'system:admin' user.

In case you use a shell having the `KUBECONFIG` variable set and where you currently have 'system:admin' as the user for ```oc whoami``` being set as user context, if you execute:

```sh
[root@bastion ~]# oc login -u <username> -p <password>  https://api.ocp4.hX.rhaw.io:6443
```

this will cause an error like this:

```sh
error: x509: certificate signed by unknown authority
```

This error is known and there is a solution for this: [https://access.redhat.com/solutions/4505101](https://access.redhat.com/solutions/4505101)

To solve it we need to list all our oauth-openshift-pods

```sh
[root@bastion ~]# oc get pods -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-5bffc98df5-c7jzh   1/1     Running   0          34m
oauth-openshift-5bffc98df5-z7d8n   1/1     Running   0          34m
```

Now we select the first pod in our list and execute the following command:

```sh
[root@bastion ~]# oc rsh -n openshift-authentication <oauth-openshift-pod> cat /run/secrets/kubernetes.io/serviceaccount/ca.crt > ingress-ca.crt
```

Now execute the login command adding the `--certificate-authority` option:

```sh
[root@bastion ~]# oc login -u <username> -p <password> https://api.ocp4.hX.rhaw.io:6443 --certificate-authority=ingress-ca.crt
```

Note that we now have modified/extended the `$KUBECONFIG` file by the additional login context. The examples below use 'the-example-user' as '<username>'.

```sh
[root@bastion ~]# oc config current-context
/api-ocp4-hX.rhaw.io:6443/the-example-user
```

You can use `oc config get-contexts` to list all available contexts.

You can switch contexts and thus the logged-in user as follows:

```sh
[root@bastion ~]# oc whoami
the-example-user
```

```sh
[root@bastion ~]# oc config use-context admin
Switched to context "admin".
```

```sh
[root@bastion ~]# oc whoami
system:admin
```

```sh
[root@bastion ~]# oc config use-context /api-ocp4-hX.rhaw.io:6443/the-example-user
Switched to context "/api-ocp4-hX.rhaw.io:6443/the-example-user".
```

```sh
[root@bastion ~]# oc whoami
the-example-user
```

## Example: ldap identity provider

> The OCP4 LDAP identity provider setup is different to setup of the provider in OCP3!

First we have to create a secret with the bindPassword of the LDAP server as content.

```sh
[root@bastion ~]# oc create secret generic ldap-secret --from-literal=bindPassword=<password> -n openshift-config
```

If possible, you must obtain the certificate authority (CA) certificate used to sign the AD server certificate. Ask your LDAP or AD administrator to provide this for you in PEM format. If this isn’t possible and if you are reasonably sure your network connection isn’t compromised, you can use openssl to retrieve the server certificate from the server. The following example demonstrates how to do this.

```sh
[root@bastion ~]# openssl s_client -connect ldap.domain.tld:636 -showcerts < /dev/null
```

If the PEM was made on a Windows machine we have to correct the line end so the PEM will work without any problems. To fix the line end issue we ran the following command against the provided file:

```sh
[root@bastion ~]# awk '{ sub("\r$", ""); print }' windows.pem > unix.pem
```

Identity providers use OpenShift Container Platform ConfigMaps in the openshift-config namespace to contain the certificate authority bundle. These are primarily used to contain certificate bundles needed by the identity provider.
Define an OpenShift Container Platform ConfigMap containing the certificate authority by using the following command. The certificate authority must be stored in the ca.crt key of the ConfigMap.

```sh
[root@bastion ~]# oc create configmap ca-config-map --from-file=ca.crt=/path/to/ca -n openshift-config
```

The following Custom Resource (CR) shows the parameters and acceptable values for an LDAP identity provider.

```yaml
...
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ldapidp 
    mappingMethod: claim 
    type: LDAP
    ldap:
      attributes:
        id: 
        - dn
        email: 
        - mail
        name: 
        - cn
        preferredUsername: 
        - uid
      bindDN: "" 
      bindPassword: 
        name: ldap-secret
      ca: 
        name: ca-config-map
      insecure: false 
      url: "ldap://ldap.example.com/ou=users,dc=acme,dc=com?uid" 
...
```

Apply the LDAP CR to the cluster:

```sh
[root@bastion ~]# oc apply -f ldap_cr.yaml
```

> Details can be found in the [product documentation](https://docs.openshift.com/container-platform/4.5/authentication/identity_providers/configuring-ldap-identity-provider.html).
