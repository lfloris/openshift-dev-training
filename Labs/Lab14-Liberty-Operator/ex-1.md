# Exercise 1 - Deploying Applications with Jenkins CI/CD

In this lab we'll explore how to use Liberty Operator to deploy application on OpenShift.
Sample Guest book application is used that uses IBM Liberty and Mongo DB that was used in previous example.

Liberty Operator is already installed in our environment, if it would not be installed, it would be simple operation as it is available in the Operator Hub.

## Setup projects
To get started, log into OpenShift using the Web Console, as described [here](../Getting-started/log-in-to-openshift.md).

Once you're logged in, create a new project for this deployment. Go to Home > Projects in the navigation pane, then select 'Create Project'

![](../Getting-started/img/create-project.png)

In the 'Create Project' dialogue box that appears, use the naming format `lab14-liberty-operator`. 


### Update the service account

We will use application image that we built and pushed in the previous lab.
As the image was pushed to other project namespace, we need to add required role to be able to pull image from differnt project.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

Issue the commands below to allow the jenkins service account to edit artifacts in these project:

```
oc policy add-role-to-user system:image-puller system:serviceaccount:lab14-liberty-operator:default -n lab13-jenkins
```

## Create required resources

In this lab we will deploy application creating custom resources that are observed by the operator. Guest book application is using Mongo DB, that was created in the previous lab. We will use that database in this lab also. So we will need two resources, one for Liberty application and one for Secret that will point to Mongo database.

### Create Secret for accessing Mongo
To point application to correct location and credentials for the database, it is using environment entries passed via 'Secret'. This secret was created automatically in the previous lab, but in this lab we will create it manually.

In the web console select 'Workloads > Secrets' and click 'Create' button. Select `Key/Value` secret type.
Specify secret name: `mongo-url`, Key: `mongo-url` and in the Data section type: `mongodb://mongo:mongo@mongodb.lab13-jenkins:27017/?authSource=mydb` as shown below:

![](img/mongo-secret.png)

You may also change to the YAML form and paste contents of [this file](openshift/mongo-secret.yaml):

```
kind: Secret
apiVersion: v1
metadata:
  name: mongo-url
data:
  mongo-url: >-
    bW9uZ29kYjovL21vbmdvOm1vbmdvQG1vbmdvZGIubGFiMTMtamVua2luczoyNzAxNy8/YXV0aFNvdXJjZT1teWRi
type: Opaque
```

### Create Liberty application resource

Application can be deployed via command line or web console. In this lab we will use web console.

Navigate to 'Operators > Installed Operators' and start typing `liberty` in the filter field, you should see `Open Liberty Operator`:

![](img/liberty-operator.png)

Select the operator and on the operator details page click `Create Instance` in the `Open Liberty Application` API:

![](img/liberty-operator-create.png)


Replace resource definition with [this one](openshift/liberty-app.yaml) 

```
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: guestbook-app
  namespace: lab14-liberty-operator
spec:
  replicas: 1
  applicationImage: 'lab13-jenkins/get-started-java:latest'
  serviceability:
    size: 1Gi
  expose: true
  env:
    - name: MONGO_URL
      valueFrom:
        secretKeyRef:
          name: mongo-url
          key: mongo-url
```

Click 'Create' button.

The following resources are automatically created by the Liberty Operator:

- Deployment 
- Service 
- Route

You can use web console to navigate to 'Networking > Routes' to see created route.

Click link in the 'Location' section to access the application. Remember to add `/myapp` to access aplication start page.

## Create a thead dump of the running server

Navigate to 'Operators > Installed Operators' and select again `Open Liberty Operator`

This time click `Create Instance` in the `Open Liberty Dump` API:

![](img/liberty-operator-dump.png)

The following yaml is displayed:

```
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyDump
metadata:
  name: example-dump
  namespace: lab14-liberty-operator
spec:
  podName: Specify_Pod_Name_Here
  include:
    - heap
    - thread
```

Remove the `- heap` line, as we just want the thread dump.

Replace `Specify_Pod_Name_Here` with the pod running the application. You can find it either via 'Workloads > Pods` or via command line. We will use the command line to not switch windows in the browser:

```
$oc get pods
NAME                             READY   STATUS    RESTARTS   AGE
guestbook-app-68db9684f9-lsv7k   1/1     Running   0          44m
```

Once you did all the updates to the yaml, click `Create` button.

Dump should be succcessfully created and you should see the list of the dumps. Click created dump and switch to the YAML tab.
In the `status` section you will find information about the location and filename of the dump:

```
 dumpFile: >-
    /serviceability/lab14-liberty-operator/guestbook-app-68db9684f9-lsv7k/2020-10-29_10:35:24.zip
```

In your local workstation in the Lab14-Liberty-Operator directory, create `dump` folder and go to that folder.

```
$ mkdir dump
$ cd dump
```

Copy the dump to your workstation (you need to replace pod name and filename with your values):

```
$ oc rsync guestbook-app-68db9684f9-lsv7k:/serviceability/lab14-liberty-operator/guestbook-app-68db9684f9-lsv7k/2020-10-29_10:35:24.zip .
```

Unzip the dump archive (replace filename):

```
$ unzip 2020-10-29_10:35:24.zip
```

To display thread dump type:

```
$ more javacore.*
```

## Enable tracing on the running server (optional)

This is optional, challenging part, no detailed instruction provided :-)
Using similar steps try to use Trace API to enable following trace `traceSpecification: "*=info:com.ibm.ws.webcontainer*=all"`.
To disable tracing you will need either set `disable` to `true` or delete trace resource.

Generated trace files, along with `messages.log` files, will be in the folder using format `/serviceability/NAMESPACE/POD_NAME/`



Lab completed.

