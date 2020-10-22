# Exercise 1 - Creating an OpenShift Application with ConfigMaps and Secrets

In this lab, you'll create a simple application Deployment resource and deploy it to OpenShift.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab02-mariadb
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab02/mariadb
$ cd Lab02/mariadb
```

One of the first things to think about is how we're going to store some credentials. Secrets offer a way to store encrypted sensitive application data within the OpenShift platform. We can create secrets in one of two ways:

  1. Via YAML
  2. Via `oc create secret`


To start, we'll use the YAML method. First we need to encode a literal string into a Base64 encoded string before we create the Secret resoure in OpenShift. You can make up your own string here, below is just an example.

In your terminal, encode a password string

```
$ echo -n 'mypassw0rd' | base64
bXlwYXNzdzByZA==
```

Create a file called `mariadb-secret.yaml`
​
```
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-root-pass
data:
  password: bXlwYXNzdzByZA==
```

Use `oc create -f <file>` to create the Secret

```
$ oc create -f mariadb-root-pass.yaml
secret/mariadb-root-pass created
```

The other, less time consuming way to create a secret is to use the `oc create secret` option directly. We can use flags such as `--dry-run` and `-o yaml` here to check what would be created. For example

```
$ oc create secret generic mariadb-root-pass --from-literal=password=mypassw0rd --dry-run -o yaml
apiVersion: v1
data:
  password: bXlwYXNzdzByZA==
kind: Secret
metadata:
  creationTimestamp: null
  name: mariadb-root-pass
```

Describe the Secret you created, to view some more infomation. 

```
$ oc describe secret mariadb-root-pass
Name:         mariadb-root-pass
Namespace:    userXX-lab02-mariadb
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  10 bytes
```

Note that the Data field contains the key you set in the YAML: password. The value assigned to that key is the password you created, but it is not shown in the output. Instead, the value's size is shown in its place, in this case, 16 bytes.

You can also view the contents of a secret by using a JSON based output search. For example

Get  the Base64 encoded string

```
$ kubectl get secret mariadb-root-pass -o jsonpath='{.data.password}'
bXlwYXNzdzByZA==
```

Pipe this to `base64` to get the raw decoded string
```
$ kubectl get secret mariadb-root-pass -o jsonpath='{.data.password}' | base64 --decode -
mypassw0rd
```

For a successful MariaDB installation, we need two more user credentials. We can use `oc create secret` passing in multiple key=value pairs in the same secret.

Create the following user-credentials secret

```
$ oc create secret generic mariadb-user-creds --from-literal=MYSQL_USER=user --from-literal=MYSQL_PASSWORD=userpassw0rd
secret/mariadb-user-creds created
```

ConfigMap provide us with a way to store non-sensitive information, such as environment variables or configuration files. ConfigMaps can be created in the same ways as Secrets. You can write a YAML representation of the ConfigMap manually and load it into OpenShift, or you can use the `oc create configmap` command to create it from the command line. 
​

First, create a file named `max_allowed.cnf` with the following content:
```
[mysqld]
max_connections = 151
max_user_connections = 145
thread_cache_size = 151
```

Create the ConfigMap with `oc`
```
$ create configmap mariadb-config --from-file=max_allowed.cnf
configmap/mariadb-config created
```

The manual YAML representation would be as follows

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mariadb-config
data:
  max_allowed_packet.cnf: |
    [mysqld]
    max_connections = 151
    max_user_connections = 145
    thread_cache_size = 151
```

Note the use of the `|` after the `data.<name>` section. This tells the YAML interpreter that a stream of content is following. Also note the indentation of the file. YAML is very sensitive to the use of tabs and spaces. ALWAYS use spaces in a YAML file, never tabs.
​
​
Now, we'll set up a `deployment.yaml` file to bring the Deployment specification, ConfigMap and Secret together. This will be the starting file, then later we'll add the references to the ConfigMaps and Secrets created earlier.

Create a file called `mariadb-deployment.yaml` with the following data
​
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mariadb
  name: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:10.4
        ports:
        - containerPort: 3306
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mariadb-volume
      volumes:
      - emptyDir: {}
        name: mariadb-volume
```

We have two secrets to be added to this deployment

1. mariadb-root-pass
2. mariadb-user-creds

For the mariadb-root-pass, this will be added as an envronment variable to the container. This reference goes in an `env` section within the `spec.template.spec.containers` array. Edit the `deployment.yaml` file and add the following contents under the line `name: mariadb`. The indentation in this snippet is done for you.

```
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mariadb-root-password
                key: password
```

Note that the name of the object is the name of the environment variable that is added to the container. The valueFrom field defines secretKeyRef as the source from which the environment variable will be set; i.e., it will use the value from the password key in the mariadb-root-pass Secret you set earlier.

You can also set environment variables from all key/value pairs in a Secret or ConfigMap to automatically use the key name as the environment variable name and the key's value as the environment variable's value. By using envFrom rather than env in the container spec, you can set the MYSQL_USER and MYSQL_PASSWORD from the mariadb-user-creds Secret you created earlier, all in one go.

Paste the following snippet into the `deployment.yaml` under the `env` section you just added. Again, the indentation is done for you.

```
        envFrom:
        - secretRef:
            name: mariadb-user-creds
```

Next, we need to add the mariadb-config ConfigMap to the deployment. We do this by creating an entry in the `spec.template.spec.volumes` array, and an entry in the `spec.template.spec.contaners.[mariadb].volumeMounts` array.

Add the following `volume` entry to the `deployment.yaml`

```
      - configMap:
          name: mariadb-config
          items:
            - key: max_allowed.cnf
              path: max_allowed.cnf
        name: mariadb-config
```
​
Next, add the following `volumeMounts` entry to the `deployment.yaml`

```
        - mountPath: /etc/mysql/conf.d
          name: mariadb-config
```

At this point, we should have everything we need to create a working mariadb pod

​The final `deployment.yaml` should look like the following. Compare it with your own to ensure the indentation lines up correctly.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mariadb
  name: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: mariadb:10.4
        name: mariadb
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mariadb-root-password
                key: password
        envFrom:
        - secretRef:
            name: mariadb-user-creds
        ports:
        - containerPort: 3306
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mariadb-volume
        - mountPath: /etc/mysql/conf.d
          name: mariadb-config
      volumes:
      - emptyDir: {}
        name: mariadb-volume
      - configMap:
          name: mariadb-config
          items:
            - key: max_allowed.cnf
              path: max_allowed.cnf
        name: mariadb-config
```
Create the resources in OpenShift with the `oc create` command.

```
$ oc create -f deployment.yaml
deployment.apps/mariadb created
```


You should now be able to use `oc get` commands to check the resources in the OpenShift cluster. For example, to get all deployments in the current project, use `oc get deployment`. To get all pods in the project, use `oc get pods`. These are very frequently used commands and have many variations through the use of filters and selectors, which is useful in clusters that have many applications per project.

```
$ oc get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
mariadb   0/1     1            0           13s
...
$ oc get pods
NAME                       READY   STATUS              RESTARTS   AGE
mariadb-7c4c64f8ff-2dx2k   0/1     ContainerCreating   0          26s
```

After a short time, the deployment should have 1/1 READY and the pod should be running. If you want to track the status of the deployment or pods as they update, you can use the `-w` flag. For example, `oc get pods mariadb -w`.

Following the pod status, we now see an error: `Error: secret "mariadb-root-password" not found`. Debugging pods and deployments will become second nature after spending enough time deploying container applications.

For the more 'keen-eyed', you may have already spotted the root cause of this issue. Earlier, we created a secret called `mariadb-root-password`, not `mariadb-root-pass`. so correct it, we need to update the `deployment.yaml` file with the correct secret name. Change the line `name: mariadb-root-password` to `name: mariadb-root-pass`. We'll need to redeploy it, so do so using `oc apply -f deployment.yaml`.

```
$ oc apply -f deployment.yaml 
Warning: oc apply should be used on resource created by either oc create --save-config or oc apply
deployment.apps/mariadb configured
```

There is only one key difference between `oc create` and `oc apply`. This is that `oc create` will only create new resource, where as `oc apply` will both create and update resources.

Once you have made this change, the OpenShift will terminate the old pod and recreate a new one from the updated Deployment spec.

```
$ oc get pods
NAME                       READY   STATUS        RESTARTS   AGE
mariadb-764b9bb895-db7wh   1/1     Running       0          4m47s
mariadb-7c4c64f8ff-2dx2k   0/1     Terminating   0          11m
```

We should now have a working mariadb deployment

```
$ oc get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
mariadb   1/1     1            1           13m
```
​
To check that the mariadb pod has consumed the data from the ConfigMap, we can use `oc rsh` to open a remote shell session directly to the container. This is a similar action to the `docker exec -it` command run in previous labs.
​
```
$ oc rsh <pod_name>
```

You can retrieve the pod name by using `oc get pods`
​
Check the environment variables present in the container
```
$ oc rsh mariadb-764b9bb895-db7wh
$ echo $MYSQL_USER=user
user=user
$ echo $MYSQL_PASSWORD
userpassw0rd
$ echo $MYSQL_ROOT_PASSWORD=mypassw0rd
mypassw0rd=mypassw0rd
```

When complete, delete the resources created, and delete the project.

```
$ oc delete deployment/mariadb configmap/mariadb-config secret/mariadb-root-password secret/mariadb-user-creds
deployment.extensions "mariadb" deleted
configmap "mariadb-config" deleted
secret "mariadb-root-password" deleted
secret "mariadb-user-creds" deleted

$ oc delete project userXX-lab02-mariadb
project.project.openshift.io "userXX-lab02-mariadb" deleted
```

Lab complete. Please move on to [Exposing a WebSphere Liberty application](exposing-websphere-liberty-ex-2.md)