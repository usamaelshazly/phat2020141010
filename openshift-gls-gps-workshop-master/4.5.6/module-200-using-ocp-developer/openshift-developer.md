## Using Openshift as an Developer

In this chapter, we're aiming to get you familiar with Red Hat CodeReady Workspaces.

### Deploying CodeReady Workspace

CodeReady is Red Hat's browser-based intelligent developer IDE. We'll use the
CodeReady operator to deploy CodeReady so that we can use it in our project
to develop our apps.

> IMPORTANT: Please replace *Username* with your username

- Create a new project `codeready-Username`
- As an Operator administrator, run this command before proceeding:

```
oc create clusterrole codeready-operator --resource=oauthclients --verb=get,create,delete,update,list,watch
oc create clusterrolebinding codeready-operator --clusterrole=codeready-operator --serviceaccount=codeready-Username:codeready-operator
```

- Navigate to Catalog -> OperatorHub
- Enter `CodeReady` in the search field

image::codeready-operator.png[image]

- Click the `CodeReady` operator
- Click `Install`

image::codeready-subscription.png[image]

- Click `Subscribe`

- Navigate `Installed Operators`

- Click onto `Red Hat CodeReady Workspace`

- Click `Create New`

- Add the specific `postgresImage` under `database` as shown below

```
apiVersion: org.eclipse.che/v1
kind: CheCluster
metadata:
  name: codeready
  namespace: codyready-test
spec:
  server:
    cheFlavor: codeready
    tlsSupport: false
    selfSignedCert: false
  database:
    externalDb: false
    chePostgresHostName: ''
    chePostgresPort: ''
    chePostgresUser: ''
    chePostgresPassword: ''
    chePostgresDb: ''
    postgresImage: 'registry.access.redhat.com/rhscl/postgresql-10-rhel7:1-35'
  auth:
    openShiftoAuth: false
    externalKeycloak: false
    keycloakURL: ''
    keycloakRealm: ''
    keycloakClientId: ''
  storage:
    pvcStrategy: per-workspace
    pvcClaimSize: 1Gi
    preCreateSubPaths: true
```

- Click `Create`

- Navigate to `Workloads` -> `Pods`

- You will see Pods as shown below are created

- Wait until all pods are `Running`

- Navigate to `Networking` -> `Routes`

- Click onto the URL under `LOCATION` for `codeready` route

- Click `Register`

- Enter the following information:

  * First name: Desday
  * Last name: User01
  * Email: any_email@desday.com
  * Username: *Username*
  * Password: openshift4

> IMPORTANT: Please replace *Username* with your username

- Click `Register`

- Will be prompt to create new workspace

- Enter `springboot-Username` as the name of the Workspace

- Select `springboot` from the list of stacks

- Click `Add or Import project` > Click `Git`

- Enter URL as `https://github.com/che-samples/web-java-spring-boot`

- Click `Add`

- Click `CREATE & OPEN`

- Click `Project` from the top menu and select `Update Project Configuration`

- Click `Maven`

- Click `Save`

- Click on the icon circle in red to go to manage command

- In the manage command pane open the build command folder and double click on the build file

- Scroll down to `Apply to` session

- Change `Applicable` to `Yes` by click where the red circle is

- Click `Save`

- Scroll back up to the top

- Click `RUN`

- Click `+` next to `RUN` menu on the left as show below -> double click onto `Maven`

- Enter `Build and Run` as the name

- Replace the line below in the `Command Line` area

```
cd ${current.project.path}
mvn spring-boot:run
```

- Scroll down and replace `${server.springboot}` in the `Preview URL` session

- Click `Save`

- Scroll back up and click `RUN` the green button

- Click the preview URL in the terminal (indicates in the image)

### Create factory

- Continue working on the workspace

- Click `Workspace` --> `Create Factory`

- Enter 'SpringBootSample' as name

- Click `Create` -> `Close`

- Click `Workspace` -> 'Stop'

- Click `Factory (1)` on the left menu

- Click onto `SpringBootSample`

- Scroll down and look for `Configure Actions`

- Add `Buid and Run` to `runCommand` -> Click `Add`

- Click `Open`

- Click `Back to Dashboard` at the bottom

- Click running workspace under `RECENT WORKSPACES` on the left menu

- Wait for the workspace to come up

- You will be able to start building and running the workspace

Congratulations!! You now know how to deploy CodeReady and deploy an application.