# Exercise 1 - Deploying Polyglot application
In this lab you will deploy polyglot application (the microservices building the app are written in different languages).

![](img/noistio.svg)



## Setup projects
Perform following tasks using `oc` tool.

### Create projects
Create 2 projects, one will hold CI/CD resouces, other will contain deployed application.
Application will be built using Tekton pipelines. 

```
oc new-project bookinfo-ci
oc new-project bookinfo-dev
```

### Assign privilidges required by pipeline framework

```
oc adm policy add-scc-to-user privileged system:serviceaccount:bookinfo-ci:pipeline -n bookinfo-ci
oc adm policy add-scc-to-user privileged system:serviceaccount:bookinfo-ci:pipeline -n bookinfo-dev
oc adm policy add-role-to-user edit system:serviceaccount:bookinfo-ci:pipeline -n bookinfo-ci
oc adm policy add-role-to-user edit system:serviceaccount:bookinfo-ci:pipeline -n bookinfo-dev
```

### Create Image Streams to hold built services

```
oc create is productpage -n bookinfo-dev
oc create is details -n bookinfo-dev
oc create is ratings -n bookinfo-dev
oc create is reviews -n bookinfo-dev
```

## Create pipelines 
Each microservice has its own pipeline and resources.

### Create pipeline and resources for productpage microservice

Go to the `Lab15-Polyglot/openshift/productpage` folder.
Perform following commnads:

```
oc project bookinfo-ci
oc apply -f .
```

You should see the following messages:
```
pipeline.tekton.dev/productpage-pipeline created
pipelineresource.tekton.dev/bookinfo-git unchanged
pipelineresource.tekton.dev/productpage-image-dev created
task.tekton.dev/productpage-build-push created
task.tekton.dev/deploy-productpage-app created
```

It created pipeline, pipeline resources (git and image), and tasks (build, and deploy).
Examine the files to learn more about tekton pipelines.

Execute the pipeline through web console. 
Select `bookinfo-ci` project, go to 'Pipelines' view, click productpage-pipeline and from 'Actions select `Start`.
Switch to `Logs` view, to see logs from the build process.

After successful build you can invoke the in the browser. In web console, select `bookinfo-dev` project,  go to Networking > Routes,  and click location link for `productpage` route.
Start page of the app is displayed, click `Normal user` link at the bottom.
Newly displayed page, will not contain all the details as other services are not deployed yet.
You can revisit this page after other services will be deployed.

### Create pipeline and resources for reviews microservice

Go to the `Lab15-Polyglot/openshift/reviews` folder.
Perform following commnads:

```
oc project bookinfo-ci
oc apply -f .
```

You should see the following messages:
```
pipeline.tekton.dev/reviews-pipeline created
pipelineresource.tekton.dev/bookinfo-git unchanged
pipelineresource.tekton.dev/reviews-image-dev created
task.tekton.dev/reviews-build-push created
task.tekton.dev/deploy-reviews-app created
```

It created pipeline, pipeline resources (git and image), and tasks (build, and deploy).
Examine the files to learn more about tekton pipelines.

Execute the pipeline through web console. 
Select `bookinfo-ci` project, go to 'Pipelines' view, click reviews-pipeline and from 'Actions select `Start`.
Switch to `Logs` view, to see logs from the build process.

Revisit product page to see Book reviews being displayed now.

### Create pipeline and resources for details microservice
Create `Lab15-Polyglot/openshift/details` folder.

In that folder create following files:
```
pipeline.yaml
resources.yaml
task-build.yaml
task-deploy.yaml
```
Use `productpage` folder as template, changing `productpage` to `details`.
If you are completely lost use `solution` folder :-)

Run pipeline in the similar way as above.

Revisit product page to see Book details being displayed now.

### Create pipeline and resources for ratings microservice
Create `Lab15-Polyglot/openshift/ratings` folder.

In that folder create following files:
```
pipeline.yaml
resources.yaml
task-build.yaml
task-deploy.yaml
```
Use `productpage` folder as template, changing `productpage` to `ratings`.
If you are completely lost use `solution` folder :-)

Run pipeline in the similar way as above.


## Enable ratings in reviews service (optional)
In the web console, select `bookinfo-dev` project, and go to Workloads > Deploymnts, click `review` deployment. 

Click `YAML` tab and after the `ports` at the same level add the following:

```
          env:
            - name: SERVICE_VERSION
              value: v2
            - name: ENABLE_RATINGS
              value: 'true'
```              

If you feel more comfortable with typing go to `Environment` tab, and type these manually.

After these changes reload product page in the browser and you should see ratings as stars.


Well done lab completed. :-)

