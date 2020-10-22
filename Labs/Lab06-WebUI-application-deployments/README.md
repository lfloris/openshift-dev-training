# Lab06 - Deploying Applications from the OpenShift User Interface

This lab contains 3 exercises

Ex 1 - [Creating a DeploymentConfig](creating-a-deploymentconfig-ex-1.md)


After each exercise, please clean up the resources created.


kind: DeploymentConfig
apiVersion: apps.openshift.io/v1
metadata:
  annotations:
    app.openshift.io/vcs-ref: master
    app.openshift.io/vcs-uri: 'https://github.com/sclorg/ruby-ex.git'
  name: ruby-ex-git-gio-dc
  generation: 2
  namespace: default
  labels:
    app: ruby-ex-git-gio-dc
    app.kubernetes.io/component: ruby-ex-git-gio-dc
    app.kubernetes.io/instance: ruby-ex-git-gio-dc
    app.kubernetes.io/name: ruby
    app.kubernetes.io/part-of: ruby-ex-git-app
    app.openshift.io/runtime: ruby
    app.openshift.io/runtime-version: '2.5'
spec:
  strategy:
    type: Rolling
    rollingParams:
      updatePeriodSeconds: 1
      intervalSeconds: 1
      timeoutSeconds: 600
      maxUnavailable: 25%
      maxSurge: 25%
    resources: {}
  triggers:
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - ruby-ex-git-gio-dc
        from:
          kind: ImageStreamTag
          namespace: default
          name: 'ruby-ex-git-gio-dc:latest'
        lastTriggeredImage: >-
          image-registry.openshift-image-registry.svc:5000/default/ruby-ex-git-gio-dc@sha256:ce05d5c42afd0dc590b567eaab8921e8413a1062f7b34aea70c04a2a41258145
    - type: ConfigChange
  replicas: 1
  test: false
  selector:
    app: ruby-ex-git-gio-dc
    deploymentconfig: ruby-ex-git-gio-dc
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ruby-ex-git-gio-dc
        deploymentconfig: ruby-ex-git-gio-dc
    spec:
      containers:
        - name: ruby-ex-git-gio-dc
          image: >-
            image-registry.openshift-image-registry.svc:5000/default/ruby-ex-git-gio-dc@sha256:ce05d5c42afd0dc590b567eaab8921e8413a1062f7b34aea70c04a2a41258145
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler