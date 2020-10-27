# Exercise 2 - Deploying an Application from the OpenShift Image Registry

In this lab we'll explore how to deploy an application from a custom image we push to the OpenShift Image Registry. This exercise contains minimal guidance.

To get started, log into OpenShift using the CLI or User Interface, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this exercise.

```
$ oc new-project lab12-image-reg-deploy
```

Using the application source code in [jpetstore](jpetstore/), create a frontend and a backend application. 

## Creating Images
The frontend Dockerfile is in the root [jpetstore](jpetstore/) directory, and the backend Dockerfile is in the [jpetstore/db](jpetstore/db/) directory. Create new container images based on this code and push these images to the OpenShift image registry.

## Creating the Applications
Create two new deployments using the images you pushed to the image registry in the previous steps. Use the below specification

### Frontend
- Use the name `jpetstoreweb` for the DeploymentConfig
- Set an `ImageChange` trigger on the DeploymentConfig to trigger a new rollout when the `latest` image tag is updated
- Use 1 replica
- Use the labels `app=jpetstoreweb`
- Use the environment variable `VERSION` with the value of `1`
- Use the `containerPort` 9080
- Use a readiness probe using `httpGet` with the `path` as `/` and the `port` as `9080`. Set the `initialDelaySeconds` to `10` and the `periodSeconds` to `5`
- Expose the deployment with a `port` of `80` and a `targetPort` of `9080`
- Create a Route to the application

### Backend
- Use the name `jpetstoredb` for the deployment
- Tag the image as `v1`
- Use the `containerPort` 3306
- Use the following environment variables
    - MYSQL_ROOT_PASSWORD="foobar"
    - MYSQL_DATABASE="jpetstore"
    - MYSQL_USER="jpetstore"
    - MYSQL_PASSWORD="foobar"
- Expose the deployment with a `port` of `3306` and a `targetPort` of `3306`
- Use a PersistentVolume mounted to the path `/var/lib/mysql`


Once completed, you should be able to access the application using the hostname provided in the Route.