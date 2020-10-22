# Exercise 1 - Deploying an Operator

In this lab you'll explore how the cluster administrator can make a PostgreSQL Operator available from the OperatorHub to other users in the cluster so they can install PostgreSQL applications managed by the MongDB Operator.

This lab will be executed in three parts

1. Use the developer to create the project
2. Use the Cluster Administrator to install the Operator
3. Use the developer to create the application from the installed Operator

## Part 1 - Create the Project

First we need to log in with a non-admin role and create a project similar to `userXX-lab08-operator`, replacing `userXX` with your user ID.

Log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

Create the project

```
$ oc new-project userXX-lab08-operator
```

## Part 2 - Install the Operator

Now we should take the role of the Cluster Administrator to make the PostgreSQL Operator available.

Log in as the admin user, following the same process as described [here](../Getting-started/log-in-to-openshift.md), except this time we log in with admin credentials. In this lab environment, use the following

```
$ oc login https://api.demo.ibmdte.net:6443 -u ibmadmin -p engageibm
```

From the navigation pain, select Operators > OperatorHub

![](img/operator-hub-view.png)

In the search bar, enter `postgresql` and then select the PostgreSQL Operator

![](img/operator-hub-postgresql.png)

Select 'Install'

![](img/operator-hub-postgresql-install.png)

On the Subscription page, select the option for 'A specific namespace in the cluster', then select the namespace you created in Part 1.

![](img/postgresql-subs.png)

Select 'Subscribe'

The PostgreSQL Operator should begin installing to the selected project. 

![](img/postgresql-installed.png)

This may take a short amount of time, so while we wait we can now log out of the Web Console with the `ibmadmin` user and log in again with our developer user.

## Part 3 - Install a Database from the PostreSQL Operator

Now log back into the Web Console as the developer, using your own user ID as described [here](../Getting-started/log-in-to-openshift.md).

From the project selection box, select the project you created at the start of this lab, for example, `user99-lab08-operators`. In the navigation pane, select Operators > Installed Operators, then select the PostgreSQL Operator. You may need to use the search box if you have a lot of Operators installed to this project.

![](img/select-postgresql-operator.png)

Select the 'Database Database' tab.

![](img/select-database-database.png)

This page will allow you to create a new instance of a PostgreSQL database. The YAML provided is perfectly valid to deploy as it is, so we do not need to make any changes other than changing the `name` field to include our username. Use the below YAML as an example.

```
apiVersion: postgresql.dev4devs.com/v1alpha1
kind: Database
metadata:
  name: user99-database
  namespace: user99-lab08-operators
spec:
  databaseCpu: 30m
  databaseCpuLimit: 60m
  databaseMemoryLimit: 512Mi
  databaseMemoryRequest: 128Mi
  databaseName: example
  databaseNameKeyEnvVar: POSTGRESQL_DATABASE
  databasePassword: postgres
  databasePasswordKeyEnvVar: POSTGRESQL_PASSWORD
  databaseStorageRequest: 1Gi
  databaseUser: postgres
  databaseUserKeyEnvVar: POSTGRESQL_USER
  image: centos/postgresql-96-centos7
  size: 1
```

Replace `user99` in the `name` field with your own user ID for this lab.

![](img/create-database.png)

When ready, select 'Create'.

This should begin the deployment process and we can check the state of our new database pods.

In the navigation pane, go to Workloads > Deployments then select the `userXX-database`, replacing `userXX` with your own user ID, or the name you gave to your database application in the previous step.

![](img/select-deployment.png)

Here, we can select the 'Pods' tab and see if our PostgreSQL database pod is running.

![](img/postresql-pod-running.png)

We have successfully installed an Operator in a developer project and the developer user has created a new application from this Operator.

Lab complete.

When you're finished, please clean up the resources created and delete the project.