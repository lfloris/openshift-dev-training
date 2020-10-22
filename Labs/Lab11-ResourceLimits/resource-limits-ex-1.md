# Exercise 1 - Using Resource Constraints

In this lab you'll create and enforce a set of quotas on a project to prevent the applications deployed to that project from breaching a set of resource constraints that we will specify.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab11-res-con
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab11/resource-constraints
$ cd Lab11/resource-constraints
```

For this lab, you will need to log in to OpenShift with cluster administrative right to enabled the use of SCCs. Please contact the lab administrator to provide you with a valid token to use throughout this lab.

After a resource quota for a project is first created, the project restricts the ability to create any new resources that may violate a quota constraint until it has calculated updated usage statistics.

After a quota is created and usage statistics are updated, the project accepts the creation of new content. When you create or modify resources, your quota usage is incremented immediately upon the request to create or modify the resource.

When you delete a resource, your quota use is decremented during the next full recalculation of quota statistics for the project. A configurable amount of time determines how long it takes to reduce quota usage statistics to their current observed system value.

If project modifications exceed a quota usage limit, the server denies the action, and an appropriate error message is returned to the user explaining the quota constraint violated, and what their currently observed usage statistics are in the system.

For this example, we're going to enforce a the following constraints

- The total number of pods in a non-terminal state that can exist in the project.
- Across all pods in a non-terminal state, the sum of CPU requests cannot exceed 1 core.
- Across all pods in a non-terminal state, the sum of memory requests cannot exceed 1Gi.
- Across all pods in a non-terminal state, the sum of ephemeral storage requests cannot exceed 2Gi.
- Across all pods in a non-terminal state, the sum of CPU limits cannot exceed 2 cores.
- Across all pods in a non-terminal state, the sum of memory limits cannot exceed 2Gi.
- Across all pods in a non-terminal state, the sum of ephemeral storage limits cannot exceed 4Gi.
 
 We can do this with the following specification

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1" 
    requests.memory: 1Gi 
    requests.ephemeral-storage: 2Gi 
    limits.cpu: "2" 
    limits.memory: 2Gi
```

Save this to a file called `compute-resources.yaml`, then create the ResourceQuota using `oc create -f compute-resources.yaml`

```
$ oc create -f compute-resources.yaml
resourcequota/compute-resources created
```

Now let's see what happens when we try to create a few `hello-openshift` deployments. 

Create a `pod.yaml` from the following content

```
apiVersion: v1
kind: Pod
metadata:
  name: lab07-ex-1-
spec:
  containers:
  - name: lab07
    image: polinux/stress
    resources:
      limits:
        memory: 200Mi
        cpu: 200m
      requests:
        memory: 100Mi
        cpu: 500m
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

This is a very basic pod, but it's suitable to showcase the ResourceQuota here

Deploy it to OpenShift

```
$ oc create -f pod.yaml
pod/lab011-ex-1-nhvl4 created
```

Because we used the `generateName` feature in the pod spec combined with `oc create`, we can keep redeploying the pod without having to edit the file and specify a new name. Repeat this step 4 times

```
$ oc create -f pod.yaml
pod/lab011-ex-1-wp8ts created
```

On the fourth attempt, OpenShift shuld block us from creating more pods because we have gone over quota.

```
$ oc get pods
NAME                READY   STATUS    RESTARTS   AGE
lab011-ex-1-cddsb   1/1     Running   0          37s
lab011-ex-1-nhvl4   1/1     Running   0          2m43s
lab011-ex-1-p7nln   1/1     Running   0          50s
lab011-ex-1-wp8ts   1/1     Running   0          96s

$ oc create -f pod.yaml 
Error from server (Forbidden): error when creating "pod.yaml": pods "lab011-ex-1-jmhp5" is forbidden: exceeded quota: compute-resources, requested: limits.cpu=500m,pods=1, used: limits.cpu=2,pods=4, limited: limits.cpu=2,pods=4

```

We can also try this exercise using Deployments, and any other type of resource that generates pods. Create a file called `deployment.yaml` with the following content

```
apiVersion: apps/v1
kind: Deployment
metadata:
  generateName: lab11-deploy-
spec:
  selector:
    matchLabels:
      app: hello-openshift
  replicas: 2
  template:
    metadata:
      labels:
        app: hello-openshift
    spec:
      containers:
        - name: hello-openshift
          image: openshift/hello-openshift
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 200m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

Deploy this

```
$ oc create -f deployment.yaml
deployment.apps/lab11-deploy-dqgpn created
```

Now check the currently deployedd pods

```
$ oc get pods
NAME                READY   STATUS    RESTARTS   AGE
lab011-ex-1-cddsb   1/1     Running   0          5m1s
lab011-ex-1-nhvl4   1/1     Running   0          7m7s
lab011-ex-1-p7nln   1/1     Running   0          5m14s
lab011-ex-1-wp8ts   1/1     Running   0          6m
```

Despite creating deployments with 2 replicas defined, the scheduler will not create any new pods until we are under quota. Whilst it's not completely obvious at first, since we're expecting 6 pods and only see 4, we can check the events to look for errors.

```
$ oc get events
LAST SEEN   TYPE      REASON              OBJECT                          MESSAGE
104s        Warning   FailedCreate        replicaset/lab11-deploy-dqgpn-f4894b6b8   Error creating: pods "lab11-deploy-dqgpn-f4894b6b8-k4ftl" is forbidden: exceeded quota: compute-resources, requested: limits.cpu=500m,pods=1, used: limits.cpu=2,pods=4, limited: limits.cpu=2,pods=4
104s        Warning   FailedCreate        replicaset/lab11-deploy-dqgpn-f4894b6b8   Error creating: pods "lab11-deploy-dqgpn-f4894b6b8-qzm26" is forbidden: exceeded quota: compute-resources, requested: limits.cpu=500m,pods=1, used: limits.cpu=2,pods=4, limited: limits.cpu=2,pods=4
```

So now we know why we don't have 6 pods - we have exceeded the quota. This same process can apply to a range of different resource types. More information andd examples can be found at https://docs.openshift.com/container-platform/4.3/applications/quotas/quotas-setting-per-project.html

When ready, remove the resources created and the project.

Lab complete.