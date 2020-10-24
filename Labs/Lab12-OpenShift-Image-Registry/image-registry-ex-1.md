# Exercise 1 - Using the OpenShift Image Registry

In this lab we'll explore how to use the OpenShift Image Registry to push and pull images.

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

Pull an image to test with
```
$ docker pull python:slim-2.7
```

Tag the image to match our image registry endpoint and project. The format is `[IMAGE_REGISTRY_ROUTE]/[PROJECT]/[IMAGE_NAME]:[TAG]`
```
$ docker tag python:slim-2.7  ${HOST}/lab12-image-reg/my-python:slim-2.7
```

Push the image to the registry
```
$ docker push ${HOST}/lab12-image-reg/my-python:slim-2.7
```

Now we can try pulling it again to make sure it's uploaded

```
$ docker pull ${HOST}/lab12-image-reg/my-python:slim-2.7
```

Lab complete. Please move on to [Deploying an Application from the Image Registry](app-deploy-registry-image-ex-2.md)