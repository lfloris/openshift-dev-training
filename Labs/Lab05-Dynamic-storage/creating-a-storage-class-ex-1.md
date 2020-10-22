# Exercise 1 - Creating a Storage Class

In this lab you'll create a Storage Class that you'll later use to dynamically create Persistent Volumes from.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab05-sc
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab05/storage-class
$ cd Lab05/storage-class
```

Creating a Storage Class in OpenShift is pretty straight forward. All that OpenShift needs is a YAML file defining the type of storage that is accessible in your environment. In this lab environment, an NFS storage provisioner has already been defined that can dynamically create volumes from new PVCs. There are a lot of supported storage provisioners from a multitude of providers so it entirely depends on your infrastructure as to which ones to use. One of the benefits of OpenShift is that if the desired storage has a valid Container Storage Interface plugin, it can be integrated with OpenShift as a container storage provider.

Using the CLI, create a new file called `nfs-sc.yaml` with the following content, replacing `user99` with your own user ID.

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: user99-dynamic-nfs-storage
provisioner: nfs-provisioning
parameters:
  archiveOnDelete: 'false'
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

Create the resource

```
$ oc create -f nfs-sc.yaml
storageclass.storage.k8s.io/user99-dynamic-nfs-storage created
```

Check the storage class was created
```
$ oc get storageclass
NAME                            PROVISIONER        AGE
managed-nfs-storage (default)   nfs-provisioning   100d
user99-dynamic-nfs-storage      nfs-provisioning   73s
```

Now you should be able to consume this storage class in a PVC.

Create a new PVC, using the UI or CLI as in previous exercises, this time using the new storage class. Use the below example as a reference

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: user99-pvc-dynamic-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      user: user99
  storageClassName: "user99-dynamic-nfs-storage"
```

Once created, the PVC should state `Bound`

![](img/dynamic-pvc-created-ui.png)

When you're finished, delete the created PVC and the project. We will reuse the Storage Class in the next exercise.

Lab Complete. Please move on to [Dynamic Storage with StatefulSets](dynamic-storage-statefulset-ex-2.md)