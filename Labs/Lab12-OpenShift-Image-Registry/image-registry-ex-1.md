# Exercise 1 - Using the OpenShift Image Registry

In this lab we'll explore how to push local images to our OpenShift Image Registry to use in an application. In this exercise we'll locally build a Python image as a base, create a new application from that image and push that image to the OpenShift Image Registry.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project lab12-image-reg
```

Get the image registry endpoint from OpenShift. This is a Route in the `openshift-image-registry` project.
```
$ HOST=$(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')
```

Log in to the registry
```
$ docker login -u $(oc whoami) -p $(oc whoami -t) ${HOST}
Login succeeded!
```

Build the `my-python` image that was used in [Lab01](../Lab01-Building-a-container/custom-docker-image-ex-4.md). You may already have this image locally built, so you can use that instead of repeating that lab exercise. For this example, the local image name is `mypython` and for the tag we'll use `latest`.

Tag the image to match our image registry endpoint and project. The format is `[IMAGE_REGISTRY_ROUTE]/[PROJECT]/[IMAGE_NAME]:[TAG]`
```
$ docker tag mypython:latest  ${HOST}/lab12-image-reg/mypython:latest
```

Push the image to the registry
```
$ docker push ${HOST}/lab12-image-reg/mypython:latest
The push refers to repository [default-route-openshift-image-registry.apps.demo.ibmdte.net/lab12-image-reg/mypython]
770af248d87e: Pushed 
80961999fe51: Pushed 
6bcfb40de1f0: Pushed 
b1f0ac2db4af: Pushed 
43ab60ec448a: Pushed 
b827ad64bbf6: Pushed 
5dacd731af1b: Pushed 
latest: digest: sha256:80c3acca19257e5118afdd082907b8450d02f7e0188d9d3e993e5fee8b691bf8 size: 1788
```

Now we can try pulling it again to make sure we're able to pull it from the OpenShift Image Registry

```
$ docker pull ${HOST}/lab12-image-reg/mypython:latest
latest: Pulling from lab12-image-reg/mypython
Digest: sha256:80c3acca19257e5118afdd082907b8450d02f7e0188d9d3e993e5fee8b691bf8
Status: Image is up to date for default-route-openshift-image-registry.apps.demo.ibmdte.net/lab12-image-reg/mypython:latest
```

Images pushed to the built-in registry automatically get an ImageStream created for us. We can use this ImageStream to reference our image from the registry without specifying the full image name with the host URL. We can view this using `oc get imagestreams`

```
$ oc get imagestreams
NAME       IMAGE REPOSITORY                                                                       TAGS        UPDATED
mypython   default-route-openshift-image-registry.apps.demo.ibmdte.net/lab12-image-reg/mypython   latest      About a minute ago
```

Now we can create a new application based on the image we just pushed. For this, we'll use a shortcut available from the `oc` cli to create a new DeploymentConfig from the image name. First, since we know this particular image requires the `anyuid` SCC, apply that to the default ServiceAccount

```
$ oc adm policy add-scc-to-user anyuid -z default
securitycontextconstraints.security.openshift.io/anyuid added to: ["system:serviceaccount:lab12-image-reg:default"]
```

Create the DeploymentConfig using the `mypython:latest` image
```
$ oc create deploymentconfig mypython --image=mypython:latest
deploymentconfig.apps.openshift.io/mypython created
```

To enable the DeploymentConfig to find the image from the ImageStream, we need to enable image lookups on the `mypython` DeploymentConfig

```
$ oc set image-lookup dc/mypython
deploymentconfig.apps.openshift.io/mypython image lookup updated
```

Now check that the pods are running

```
$ oc get pods
NAME                READY   STATUS      RESTARTS   AGE
mypython-2-deploy   0/1     Completed   0          3m
mypython-2-lghlp    1/1     Running     0          2m57s
```

Check the DeploymentConfig `mypython` using `oc get deploymentconfig mypython -o yaml` and look at the `triggers` field. It's currently only set to `type: ConfigChange`. This means that if there are any changes to the actual deployment specification, a new set of pods will be rolled out. in most cases this is fine, but during development we might want to enable automated rollouts of a new image version when I push a new image to the registry. We can do this by adding a new `trigger` to the DeploymentConfig. Thee `oc` cli provides an easy way to add this trigger by using `oc set triggers` and providing the DeploymentConfig name, the imagestream and the container name.
```
$ oc set triggers deploymentconfig/mypython --from-image=mypython:latest -c default-container
deploymentconfig.apps.openshift.io/mypython triggers updated
```

Now check the DeploymentConfig again. This time the `triggers` section has a new configuration

```
  - imageChangeParams:
      automatic: true
      containerNames:
      - default-container
      from:
        kind: ImageStreamTag
        name: mypython:latest
        namespace: lab12-image-reg
      lastTriggeredImage: image-registry.openshift-image-registry.svc:5000/lab12-image-reg/mypython@sha256:80c3acca19257e5118afdd082907b8450d02f7e0188d9d3e993e5fee8b691bf8
    type: ImageChange
```

Now each time we push a new version of the `mypython:latest` image, the DeploymentConfig should rollout a new version of our application

The following steps demonstrate how this works

```
#get the current pods

$ oc get pods
NAME                READY   STATUS      RESTARTS   AGE
mypython-2-deploy   0/1     Completed   0          6m57s
mypython-4-6b8w4    1/1     Running     0          61s
mypython-4-deploy   0/1     Completed   0          64s

# build, create and push a new version of the mypython application with 3.6-slim instead of 2.7-slim

$ vim app.py 
$ vim Dockerfile

# remove (or retag) the old version of "latest"

$ docker rmi ${HOST}/lab12-image-reg/mypython:latest
Untagged: default-route-openshift-image-registry.apps.demo.ibmdte.net/lab12-image-reg/mypython:latest
Deleted: sha256:b23673eb4c3519e820570d22642b8b6d4b08b817cddeff614be4a5a9b73d9bda
Deleted: sha256:60e4b27445e17d7e51e3c42696f23033bb481fea07ddd73e40bc556bcdcb6a5b
Deleted: sha256:e67d0fe2650498b2bb2493ebeb4055e7098b849264e622e16a7fef909b8ecccb
Deleted: sha256:8cc73812db7823ed7eb387b2c0549916279f0d9eafb6afdbd35236194150ed0a
Deleted: sha256:1ca9584950825f77ac58d5788972930916698715883fe9ca998d413f48ed4c8b
Deleted: sha256:5b2d9cfac7988233e79837f7506998b7b903961d4a82de4333e2a5d31b55ce67
Deleted: sha256:18c4831b54a08a3ab1c53a05ec719c4c7036c9d6efee7670659105165b3c4593
Deleted: sha256:1baca8c6dbbaa3eb8ff773541a92c6da912e463a9193c8f9fae110b525254cd1
Deleted: sha256:a47ecf344660d8ae1b38bb79b94bf41559be477b173127e511e307d87ad1eb4c


# build the new image tagged directly with the openshift image registry name

$ docker build -t ${HOST}/lab12-image-reg/mypython:latest . --no-cache
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM python:2.7-slim
....
Successfully built 948fe2682fe4
Successfully tagged default-route-openshift-image-registry.apps.demo.ibmdte.net/lab12-image-reg/mypython:latest

# push the image to the openshift image registry
$ docker push ${HOST}/lab12-image-reg/mypython:latest
...
latest: digest: sha256:4b311ceb1a8cc3482fb5b50e6ab099dc35119bcbb9f00e5b2a318b8443534f01 size: 1994

# check if the new pods have been rolled out

$ oc get pods
NAME                READY   STATUS        RESTARTS   AGE
mypython-2-deploy   0/1     Completed     0          37m
mypython-4-6b8w4    1/1     Terminating   0          31m
mypython-4-deploy   0/1     Completed     0          31m
mypython-5-deploy   0/1     Completed     0          21s
mypython-5-ndzrr    1/1     Running       0          18s
```

Lab complete. Please move on to [Deploying an Application from the Image Registry](app-deploy-registry-image-ex-2.md)