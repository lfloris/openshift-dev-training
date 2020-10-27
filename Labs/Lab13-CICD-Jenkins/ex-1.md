# Exercise 1 - Deploying Applications with Jenkins CI/CD

In this lab we'll explore how to use Jenkins to build and deploy application on OpenShift.
Sample Guest book application is used that uses IBM Liberty and Mongo DB. 

## Setup project
To get started, log into OpenShift using the Web Console, as described [here](../Getting-started/log-in-to-openshift.md).

Once you're logged in, create a new project for this deployment. Go to Home > Projects in the navigation pane, then select 'Create Project'

![](../Getting-started/img/create-project.png)

In the 'Create Project' dialogue box that appears, use the naming format `lab13-jenkins`. Completing the Display Name and Description fields are recommended, but optional.

![](img/create-project-dialog-ex-1.png)

## Deploy Mongo DB
