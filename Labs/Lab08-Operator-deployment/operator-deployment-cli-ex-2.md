# Exercise 2 - Deploying an Operator from the CLI

In this exercise, you'll learn how to deploy an operator using the CLI only. This exercise contains no guidance on what resources are required, you'll have to figure that out based on [Exercise 1](operator-deployment-we-ex-1.md).

The operator you will need to deploy in this exercise is the **Redis Enterprise Operator** from the Certified Operators catalog.

## Hints
Use some of the `oc` commands from the previous exercise to discover the correct resources, channels and versions to deploy.

This operator can be tricky to debug, as it requires additional resources such as Security Context Constraints to run properly. Since SCCs will be covered in a later topic, use the below commands to retrieve a valid SCC from the RedisLabs GitHub and create it

```
$ curl -O https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/master/openshift/scc.yaml

$ oc apply -f scc.yaml
securitycontextconstraints.security.openshift.io/redis-enterprise-scc created
```

You'll then need to apply this SCC to the Service Account created during deployment of the RedisEnterpriseCluster resource. Use the below command as an example

```
oc adm policy add-scc-to-user redis-enterprise-scc -z <service-account-name>
```

You can expose the Redis Enterprise UI using an OpenShift Route. Use the below as a template for an SSL Passthrough (simply using `oc expose <svc>` on the UI service does not work in this scenario, we need a more complex route configuration)

```
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: redis-ui
  namespace: lab08-operators
spec:
  to:
    kind: Service
    name: <service-name>
    weight: 100
  port:
    targetPort: ui
  tls:
    termination: passthrough
    insecureEdgeTerminationPolicy: Redirect
```