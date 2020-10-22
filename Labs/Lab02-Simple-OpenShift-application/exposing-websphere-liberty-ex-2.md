# Exercise 2 - Creating and Exposing a WebSphere Liberty OpenShift Application

In this lab, you'll create a simple WebSphere Liberty application Deployment resource and deploy it to OpenShift. The application resource consumption will be limited using the `resources` spec, the application will target a specific node, and the application will be exposed so it is accessible outside of the cluster.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab02-was
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab02/was
$ cd Lab02/was
```

Create a file called `ws-deployment.yaml` with the following data to define the Deployment specification
​
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websphere-liberty-app
spec:
  selector:
    matchLabels:
      app: websphere-liberty-app
  replicas: 1
  template:
    metadata:
      labels:
        app: websphere-liberty-app
    spec:
      containers:
      - name: websphere-liberty-app
        image: websphere-liberty:19.0.0.9-webProfile8-java11
        ports:
          - containerPort: 9080
        volumeMounts:
          - mountPath: "/config/dropins"
            name: was-persistence
      volumes:
        - name: was-persistence
          emptyDir: {}
```
​
Create the resources in OpenShift with the `oc create` command. You can do this on an entire directory too.
```
$ oc create -f ws-deployment.yaml
deployment.apps/websphere-liberty-app created
```
​
You should now be able to use `oc get` commands to check the resources in the OpenShift cluster. For example, to get all deployments in the current project, use `oc get deployment`. To get all pods in the project, use `oc get pods`. These are very frequently used commands and have many variations through the use of filters and selectors, which is useful in clusters that have many applications per project.

```
$ oc get deploy
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
websphere-liberty-app   0/1     1            0           17s

$ oc get pods
NAME                                     READY   STATUS              RESTARTS   AGE
websphere-liberty-app-556bdf88b6-bzn9x   0/1     ContainerCreating   0          15s
```

After a short time, the deployment should have 1/1 READY and the pod should be running.

```
$ oc get deploy
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
websphere-liberty-app   1/1     1            1           73s

$ oc get pods
NAME                                     READY   STATUS    RESTARTS   AGE
websphere-liberty-app-556bdf88b6-bzn9x   1/1     Running   0          71s
```
​
So now  WebShpere Liberty is deployed without any restrictions in terms of resources, we should add some resource limits and resource requests to the deployment. We can do this using the `resources` specification, which looks like the following

```
resources:
  requests
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 200m
    memory: 512Mi
```

Edit your current deployment using `oc edit deployment websphere-liberty-app`. This wll open an editor in the termiinal, similar to `vi`.

Add the resource defined above to the `spec.template.spec.containers.[websphere-liberty-app]` so it looks similar to the following

```
spec:
  selector:
    matchLabels:
      app: websphere-liberty-app
  replicas: 1
  template:
    metadata:
      labels:
        app: websphere-liberty-app
    spec:
      containers:
      - name: websphere-liberty-app
        image: websphere-liberty:19.0.0.9-webProfile8-java11
        resources:
          limits:
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 128Mi
```

Save and close the editor, typically done by hitting the `esc` key on your keyboard, followed by `:wq`.

OpenShift should now roll out a new pod with the new resource requests and limits. You can view detailed pod information by using `oc describe pod <name>`. Retrieve the pod name using `oc get pods`.

```
$ oc get pods
NAME                                     READY   STATUS    RESTARTS   AGE
websphere-liberty-app-66c9f5db46-rz7tt   1/1     Running   0          76s

$ oc describe pods websphere-liberty-app-66c9f5db46-rz7tt
<additional content>
Containers:
  websphere-liberty-app:
    Container ID:   cri-o://fa1b0469c8362f0fbabf6187bc449cc955a45f7ca11d74fe463dfa13916386c2
    Image:          websphere-liberty:19.0.0.9-webProfile8-java11
    Image ID:       docker.io/library/websphere-liberty@sha256:ae0962d601fa3f11705d0a75355fa59f1f9636974d89311b7aa1845a92497df2
    Port:           9080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 28 Jun 2020 05:56:05 -0700
    Ready:          True
    Restart Count:  0
    Limits:                         << NOTE THE NEW RESOURCE LIMITS ARE ADDED
      cpu:     200m
      memory:  256Mi
    Requests:
      cpu:     100m
      memory:  128Mi
    Environment Variables from:
      ws-config   ConfigMap  Optional: false
      ws-secret   Secret     Optional: false
    Environment:  <none>
    Mounts:
      /config/dropins from was-persistence (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8dk9d (ro)
<additional content>
```

A simple way to achieve this from the CLI is also given below. You do not need to run this, as it will set the same resources as above, it is just here for reference.

```
$ oc set resources deployment websphere-liberty-app -c=websphere-liberty-app --limits=cpu=200m,memory=256Mi --requests=cpu=100m,memory=128Mi
deployment.extensions/websphere-liberty-app resource requirements updated
```

Now that the application has some defined resources, next we'll ensure the application will only be deployed to specific worker nodes in the cluster.

Let's view on which node the pod is currently deployed. Using the `-o wide` flag enables addtional fields to display.

```
$ oc get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE    IP              NODE                           NOMINATED NODE   READINESS GATES
websphere-liberty-app-66c9f5db46-rz7tt   1/1     Running   0          8m3s   10.254.15.122   worker1   <none>           <none>
```

It's currently running on worker1, since we haven't told the OpenShift Scheduler where we want it to run. So now we'll edit the application to run only on nodes will the label `app=was`.

Edit the `websphere-liberty-app` Deployment to add the below snippet to the `spec.template.spec.containers` field. 


```
nodeSelector:
  app: was
```

The `nodeSelector` should start in-line with the `volumes` section, as given below

```
spec:
  selector:
    matchLabels:
      app: websphere-liberty-app
  replicas: 1
  template:
    metadata:
      labels:
        app: websphere-liberty-app
    spec:
      containers:
      - name: websphere-liberty-app
<additional content>
      nodeSelector:
        app: was
      volumes:
        - name: was-persistence
          emptyDir: {}
```

Save and close the editor.
​
After some time, the pod should be moved to the nodes labelled with `app=was`.

```
$ oc get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                           NOMINATED NODE   READINESS GATES
websphere-liberty-app-b5679d7f8-4hhq4   1/1     Running   0          53s   10.254.0.33   worker2                        <none>           <none>
```

Now the application is successfully running where we want it. In some cases, however, it is not always ths smooth. A common scenario is that a node's resources can become totally consumed by many applications, and the scheduler can no longer schedule new workloads to that node until resources are freed. In that case, the following message is given when describing a `Pending` pod

```
  Warning  FailedScheduling  <unknown>  default-scheduler  0/8 nodes are available: 1 Insufficient memory, 2 Insufficient cpu, 7 node(s) didn't match node selector.
```

So now that the WebSphere application is deployed, we need to expose it so we can access the application from outside OpenShift. To do this, we create two resources

1. A Service
2. A Route to the Service
​
The Service will expose the specified container port and provide it with an IP and DNS name in the Service IP range. In this instance, we only have one pod but Services become more important when you have multiple replicas of an application and you expect the traffic sent to mulitple replicas to be properly load balanced and received by all relevant pods.

Create a file called `ws-svc.yaml` with the following content
​
```
apiVersion: v1
kind: Service
metadata:
  name: websphere-liberty
spec:
  ports:
  - port: 9080
  selector:
    app: websphere-liberty-app
```
​
Create the Service in OpenShift
```
oc create -f ws-svc.yaml
service/websphere-liberty created
```

View that the service was created using `oc get svc`

```
$ oc get svc websphere-liberty
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
websphere-liberty   ClusterIP   172.30.51.229   <none>        9080/TCP   12s
```
​
Notice that the Service created shows as a `ClusterIP` type. This is the default if we do not specify a specific method in the `spec.type` field. In this lab we will expose this Service through the OpenShift Router using an OpenShift Route, but in other cases you may want to expose the applcation using a different method, such as NodePort.

To easily expose the application, use the following command

```
$ oc expose service websphere-liberty
route.route.openshift.io/websphere-liberty exposed
```

You can view the new Route using the following command

```
oc get routes
NAME                HOST/PORT                                              PATH   SERVICES            PORT   TERMINATION   WILDCARD
websphere-liberty   websphere-liberty-was-test.apps.ocp4.os.fyre.ibm.com          websphere-liberty   9080                 None
```

In your web browser, navigate to the URL given in the HOST/PORT section. This should bring up the WebSphere landing page.

When you're finished, clean up the resources.

```
$ oc delete deployment/websphere-liberty-app  service/websphere-liberty route/websphere-liberty
service "websphere-liberty" deleted
route.route.openshift.io "websphere-liberty" deleted
deployments.extensions "websphere-liberty-app" deleted

$ oc delete project userXX-lab02-was
project.project.openshift.io "userXX-lab02-was" deleted
```

Lab complete.