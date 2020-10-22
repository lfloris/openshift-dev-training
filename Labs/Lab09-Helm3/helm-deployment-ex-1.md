# Exercise 1 - Deploying Applications with Helm3

This lab will demonstrate how to create, develop, publish and install a custom Helm chart to OpenShift. It will cover the basic development cycle to convert a set of Kubernetes resources to a Helm chart and manage deployment releases within OpenShift.

There is a Helm3 Templating guide available [here](../Getting-started/helm3-overview.md)

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab09-helm3
```

Replace `userXX` with your user ID or other name.

## Retrieving the Helm3 CLI

Download the Helm3 CLI tool from the following link
```
$ sudo curl -Lo /usr/local/bin/helm3 https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 && sudo chmod ua+x /usr/local/bin/helm3
```

Verify Helm3 CLI can communicate with OpenShift 

```
$ helm3 list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

Helm is commonly known as a Kubernetes package manager. It's a way to combine multiple Kubernetes resouces into a single deployable artefact that contains all the necessary Kubernetes resources to deploy an application.

## Creating a New Chart

To create a Helm chart, run the following command

```
$ helm3 create python-app
Creating python-app
```

This will create a directory called `python-app` and within it, a several new directories and files. The name of the chart created will be used in several places, so it's best to ensure you use a suitable chart name.

The following resources are created for you

| Name | Description |
| ---- | ----------- |
| charts/ |  Directory to contain all dependent charts   |
| values.yaml |  The initial values file   |
| Chart.yaml |   File containing information about the chart such as a description and versioning   |
|templates/ | Directory to contain all chart resources  |
|templates/NOTES.txt | Default file displayed post-installation that contains instructions to the user on how to use the chart |
|templates/_helpers.tpl | Initial template file used to provide templated data to other files within the chart  |
|templates/deployment.yaml |  Initial sample Kubernetes deployment  |
|templates/ingress.yaml |  Initial sample Kubernetes ingress |
|templates/service.yaml | Initial sample Kubernetes service  |
|tests/| Directory to store application post-deployment tests |
|tests/test-connection.yaml| Initial sample test file for Ngnix|

At this point, we are ready to tailor the default resources to match our application.

The default resources created by the `helm create` command may be sufficient to use with only minor adaptation. Out of the box, Helm will create a chart capable of deploying NGINX to OpenShift. We'll use these files as a basis to deploy our sample python application used in previous labs.

## Developing the Chart
Before developing the Helm chart to contain our application data, it is important that the application has already successfully been deployed to Kubernetes to ensure it works as expected. It's also important to note that this development cycle does not cover developing Docker images, as Helm charts only utilise Kubernetes resources to deploy applications based on an existing Docker image. This course has already covered simple deployments as well as validating the Dockerfile using in Lab01. 

Let's have a look at the current `python-app/deployment.yaml`

```
$ cat python-app/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "python-app.fullname" . }}
  labels:
    {{- include "python-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "python-app.selectorLabels" . | nindent 6 }}
      user: {{ .Values.myuserid }}
  template:
    metadata:
      labels:
        {{- include "python-app.selectorLabels" . | nindent 8 }}
        user: {{ .Values.myuserid }}
    spec:
    {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      serviceAccountName: {{ include "python-app.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
    {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
    {{- end }}
```

We can see that we already have a fairly standard Deployment resource pre-populated, and for the exercise of deploying our simple Python application, it is actually fairly well suited to leave it as it is as it contains all the normal best practices and a variety of deployment capabilities that we would typically use in a standard application deployment. To customise it a little for this exercise, let's see how we can add some Helm templating parameters to it. For this, we'll add an additional label to the deployed pods. We're going to do this by adding an entry to the `matchLabels` and `labels` in the Deployment and Service object using the label `user: {{ .Values.myuserid }}`, and later use the `values.yaml` file to pass in a new value for the `myuserid` parameter. Whilst this particular example is not that complex, it demonstrates how we can pass simple values from the values file (and subsequently the command line) through to a Helm template that we can repeatedly deploy as many times as we want with different values. We'll see this in action later on in the lab.

Add and entry to the end of the `spec.selector.matchLabels` list and a matching entry at the end of the `spec.selector.template.metadata.labels` list that looks like the following

```
user: {{ .Values.myuserid }}
```

The resulting code block would be this

```
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "python-app.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
      user: {{ .Values.myuserid }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "python-app.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
        user: {{ .Values.myuserid }}
```

Also change the `image` section from

```
image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"
```

to 

```
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Ensure that the indentation matches up with the other list entries.

We also need to add this selector to our `python-app/templates/service.yaml` `spec.selector`. The result looks like this

```
apiVersion: v1
kind: Service
metadata:
  name: {{ include "python-app.fullname" . }}
  labels:
    {{- include "python-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "python-app.selectorLabels" . | nindent 4 }}
    user: {{ .Values.myuserid }}
```

The `values.yaml` file is contained in the root of the application directory, and defines the values available for templating within the Helm chart. 

Edit the `python-app/values.yaml` file to add our new `myuserid` parameter and value. We're also going to modify some other parameter values in this file to deploy our simple Python application.

We need to do the following

1. Change the `image.repository` from `repository: nginx` to `quay.io/lfloris/my-python`
2. Add the following parameter `image.tag: v1` so that the code block looks like the following
```
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: v1
```
3. Add `myuserid: userXX` to the file, replacing `userXX` with your own user ID.

Once the changes have been made, our Helm application templates are now ready for testing and packaging.

Before packaging the application and deploying to OpenShift, it's a good idea to use the `helm lint` command. The Helm linter will run the application through the templating engine and look for anomolies between the values and your template definitions, and display any errors found. The run the linter, change directory to the parent directory (using `cd ..` or similar) and run `helm3 lint <chart-dir>`. If there are any errors, correct them and re-run the lint command until you receive the message `1 chart(s) linted, no failures`

Run `helm3 lint` on your application directory to check for errors.

With regards to this lab, we haven't edited a lot of content to allows us to easily make mistakes. But, for example, if you had made a mistake and missed the leading `.` from `.Values.myuserid`, the `helm3 lint` command would produce the following message

```
$ helm3 lint python-app/
==> Linting python-app/
[INFO] Chart.yaml: icon is recommended
[ERROR] templates/: parse error in "python-app/templates/service.yaml": template: python-app/templates/service.yaml:20: function "Values" not defined

Error: 1 chart(s) linted, 1 chart(s) failed
```

Otherwise, you should receive a responses similar to the following

```
$ helm3 lint python-app/
==> Linting python-app/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures
```

This means our chart is ready to deploy to OpenShift.

The deployed container image requires to be run with the root user, which requires the `anyuid` Security Context Constraint.

To enable the use of the `anyuid` SCC, you would run the following command

```
oc adm policy add-scc-to-user anyuid -z default
```

Helm offers a `--dry-run` argument that allows you to run the installer on the chart, but not actually deploy anything. This is particularly useful with the `--debug` argument that will output the contents of all the generated Kubernetes resources to stdout so you can check the resources formatting before packaging. Used in conjunction with an overriding `values.yaml` you can simulate a user deployment with custom values and check to ensure the output is correct. 

The `helm3 install` command uses the following syntax: `helm install [NAME] [CHART] [flags]`

```
$ helm3 install my-python python-app/ --dry-run --debug
```

This command should output all of the rendered resources to the terminal for us to check that the values supplied were passed correctly. If everything looks good, we can install the chart.

```
$ helm3 install my-python python-app/
NAME: my-python
LAST DEPLOYED: Tue Jun 30 15:14:14 2020
NAMESPACE: user99-lab09-helm3
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace user99-lab09-helm3 -l "app.kubernetes.io/name=python-app,app.kubernetes.io/instance=my-python" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

So the application was deployed and we can see that the pods are now running in our project

```
$ oc get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-python-python-app-9568464cc-wplrm   1/1     Running   0          50s
```

We've now installed our first Helm application on OpenShift.

To demonstrate one of the core features of Helm - the reusability - we should package our application into a reusable artifact for others to use. We can do this with `helm package`.

Run the `helm package` on your `python-app` directory

```
$ helm3 package python-app --app-version 1.0.0
Successfully packaged chart and saved it to: /home/ibmdemo/python-app-0.1.0.tgz
```

Now that we have a packaged Helm chart, we can deploy another instance of it to our cluster, but this time passing different values to the install command instead of using the default values we used in the `values.yaml` earlier. We can do this by creating our own `values.yaml` file locally, which will override any matching parameters found in the original chart values.

For example, if we wanted to use a different image tag instead of `v1`, potentially simulating another team that is deploying a stable version of our Python application, we can install the chart using the following

Before installing, ask your cluster administrator to enable the use of the `anyuid` Security Context Constraint, providing your project name.

```
$ helm3 install my-python-v1 python-app-0.1.0.tgz --set image.tag=latest
```

Once the application is deployed, we can check the image used in the pod spec

```
$ oc get pods -l app.kubernetes.io/instance=my-python-v1
NAME                                      READY   STATUS    RESTARTS   AGE
my-python-v1-python-app-554b565f5-6vbvn   1/1     Running   0          94s

$ oc describe pod my-python-v1-python-app-554b565f5-6vbvn
Name:               my-python-v1-python-app-554b565f5-6vbvn
Namespace:          user99-lab09-helm3
Priority:           0
PriorityClassName:  <none>
Node:               worker1/10.0.0.21
Start Time:         Tue, 30 Jun 2020 15:28:14 -0700
Labels:             app.kubernetes.io/instance=my-python-v1
                    app.kubernetes.io/name=python-app
                    pod-template-hash=554b565f5
Annotations:        k8s.v1.cni.cncf.io/networks-status:
                      [{
                          "name": "openshift-sdn",
                          "interface": "eth0",
                          "ips": [
                              "10.130.2.50"
                          ],
                          "dns": {},
                          "default-route": [
                              "10.130.2.1"
                          ]
                      }]
                    openshift.io/scc: anyuid
Status:             Running
IP:                 10.130.2.50
Controlled By:      ReplicaSet/my-python-v1-python-app-554b565f5
Containers:
  python-app:
    Container ID:   cri-o://be250c4bd6b6a87988c8f09246d7f2f9e9e3e130a7348bb813606c605b1281fc
    Image:          quay.io/lfloris/my-python:latest                                                                 <<<<<<<<<<<<<<<<<<< NEW IMAGE
    Image ID:       quay.io/lfloris/my-python@sha256:4256b771943b04a078bf4809f7d6741ad027b14bcf2afd1f09e27f9baeab87eb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 30 Jun 2020 15:28:19 -0700
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bdkbd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-bdkbd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bdkbd
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned user99-lab09-helm3/my-python-v1-python-app-554b565f5-6vbvn to worker1
  Normal  Pulling    2m22s      kubelet, worker1   Pulling image "quay.io/lfloris/my-python:latest"
  Normal  Pulled     2m20s      kubelet, worker1   Successfully pulled image "quay.io/lfloris/my-python:latest"
  Normal  Created    2m20s      kubelet, worker1   Created container python-app
  Normal  Started    2m19s      kubelet, worker1   Started container python-app
```

So now we have developed a working, reusable Helm based application.

When you're finished, clean up any resources created and delete the project.

Lab complete.