# Exercise 2 - Creating a StatefulSet using Dynamic Storage

In this lab you'll create a new Nginx application as a that will leverage the storage class created in previous labs.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab05-dynamic-pvc-sts
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab05/dynamic-pvc-sts
$ cd Lab05/dynamic-pvc-sts
```

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

This example will show how a StatefulSet can be coupled with dynamic storage provisioning to fully utilise scalable stateful container applications.

First, we'll create something called a Headless Service. A headless service is a service with a service IP but instead of load-balancing it will return the IPs of our associated Pods. This allows us to interact directly with the Pods instead of a proxy. It's as simple as specifying None for `.spec.clusterIP` and can be utilized with or without selectors. This becomes useful when you want to divert traffic to the individual addresses of each pod as opposed to letting the service load balance across all pods assigned to the service. For example, for a high-availability database deployment where the leading database needs to keep track of the other cluster members and send data directly to each member pod, using a normal Service will not suffice as the data will be routed to a randomly selected load-balanced pod instead of the target. 

To create the headless service, create a file called `headless-svc.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

Create the service

```
$ oc create -f headless-svc.yaml
service/web-svc created
```

Prior to deploying Nginx, it requires additional permissions to run. By default, all pods inherit the `restricted` Security Context Constraint (SCC). To enable the Nginx pods to use additional privileges required, we need to assign the `default` Service Account the `anyuid` Security Context Constraint. This subject will be covered in more depth in a later lab.

To enable the use of the `anyuid` SCC, you would run the following command

```
oc adm policy add-scc-to-user anyuid -z default
```

Next we need to deploy the StatefulSet. Below is the definition for a 3-replica Nginx web application. With the use of the `volumeClaimTemplates`, we can define specific storage requirements for the pods. It also means that each pod in the StatefulSet will create it's own indexed Persistent Volume Claim using the dynamic provisioner we created earlier. 

Create a new file `webapp-sts.yaml`

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "user99-dynamic-nfs-storage"
      resources:
        requests:
          storage: 1Gi
```

Create the StatefulSet

```
$ oc create -f webapp-sts.yaml
statefulset.apps/webapp created
```


Verify the Statefulset was created

```
$ oc get statefulset
NAME        READY   AGE
web         0/3     3s
```

Pods should start creating. After some time, they should be ready.
```
$ oc get pods
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          6m52s
web-1   1/1     Running   0          6m48s
web-2   1/1     Running   0          6m44s
```

Check the PVCs created. You should see one is created for each pod, and the name should match up with the naming convention <metadata.name>-<sts-name>-<index>.
```
$ oc get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                 AGE
www-web-0   Bound    pvc-768cf4e6-dcb6-450c-8e29-8c4d0352bd1a   1Gi        RWO            user99-dynamic-nfs-storage   10s
www-web-1   Bound    pvc-8250e684-008c-4278-b5ed-1e9dd846ca3d   1Gi        RWO            user99-dynamic-nfs-storage   6s
www-web-2   Bound    pvc-4323e8f3-82b6-474c-9f71-1ea7052b89ff   1Gi        RWO            user99-dynamic-nfs-storage   2s
```

One of the benefits of a StatefulSet with dynamic storage provisioning is that we can easily scale it and not have to worry about manually creating additional Persistent Volumes and Persistent Volume Claims.

To scale the `web` StatefulSet, use the following command
```
$ oc scale statefulset web --replicas=5
statefulset.apps/web scaled
```

We should now see that the number of desired replicas has increased to 5, and that two additional PVCs were created as a result
```
$ oc get statefulset
NAME        READY   AGE
web         0/5     3m56s

$ oc get pods
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          4m50s
web-1   1/1     Running   0          4m46s
web-2   1/1     Running   0          4m42s
web-3   1/1     Running   0          59s
web-4   1/1     Running   0          48s

$ oc get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                 AGE
www-web-0   Bound    pvc-768cf4e6-dcb6-450c-8e29-8c4d0352bd1a   1Gi        RWO            user99-dynamic-nfs-storage   6m13s
www-web-1   Bound    pvc-8250e684-008c-4278-b5ed-1e9dd846ca3d   1Gi        RWO            user99-dynamic-nfs-storage   6m9s
www-web-2   Bound    pvc-4323e8f3-82b6-474c-9f71-1ea7052b89ff   1Gi        RWO            user99-dynamic-nfs-storage   6m5s
www-web-3   Bound    pvc-63900320-0640-4425-8950-186464b7a48b   1Gi        RWO            user99-dynamic-nfs-storage   2m22s
www-web-4   Bound    pvc-a2d3c87b-1ffc-4144-9cf5-b369c2f5b0a2   1Gi        RWO            user99-dynamic-nfs-storage   2m11s
```

We can also scale this back down just as easily. The scheduler will always remove the newest pods first

```
$ oc scale statefulset web --replicas=3
statefulset.apps/web scaled

$ oc get statefulset
NAME   READY   AGE
web    3/3     8m26s

$ oc get pods
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          8m32s
web-1   1/1     Running   0          8m28s
web-2   1/1     Running   0          8m24s

$ oc get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                 AGE
www-web-0   Bound    pvc-768cf4e6-dcb6-450c-8e29-8c4d0352bd1a   1Gi        RWO            user99-dynamic-nfs-storage   8m40s
www-web-1   Bound    pvc-8250e684-008c-4278-b5ed-1e9dd846ca3d   1Gi        RWO            user99-dynamic-nfs-storage   8m36s
www-web-2   Bound    pvc-4323e8f3-82b6-474c-9f71-1ea7052b89ff   1Gi        RWO            user99-dynamic-nfs-storage   8m32s
www-web-3   Bound    pvc-63900320-0640-4425-8950-186464b7a48b   1Gi        RWO            user99-dynamic-nfs-storage   4m49s
www-web-4   Bound    pvc-a2d3c87b-1ffc-4144-9cf5-b369c2f5b0a2   1Gi        RWO            user99-dynamic-nfs-storage   4m38s
```

It is worth noting here that the PVCs created as a result of scaling horizontally were not removed. This is the default behaviour and cleaning up of PVCs is usually done by the project admin so that a PVC containing potentially production data is not automatically removed by OpenShift.

When finished, remove all of the resources you created and the project.

Lab complete.