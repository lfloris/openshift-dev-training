# Exercise 4 - Building a Customer Docker Image

In this exercise, you'll create a simple Python application image from source code built locally.

Use the resoures in the [python-image](resources/python-image) directory.

If you have cloned the entire `openshift-dev-training` Git repo, change directory to `/home/ibmdemo/ibm-ocp-training/Labs/Lab01-Building-a-container/resources/python-image`. If not, then visit the Git repository in the URL and copy the file contents directly into a new directory.

To build the image, run the following

```
$ sudo docker build -t my-python:v1 .
```

The docker image should start building, and it will be available in the list of docker images

```
$ sudo docker images my-python
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
my-python           v1                  44c13412dac4        47 seconds ago      131MB
```

We can now use this as a base image for a container deployment.

Deploy a new container from this image using exposing port 8080 on the local machine. Note that if you did not clean up containers from previous exercises, you might get errors stating port 8080 is still in use. Either remove the old container or choose a different port using the format `[HOSTPORT]:8080`.

```
$ sudo docker run -d --name my-python -p 8080:8080 my-python:v1
```

The container should now be running at port 8080, so check it by entering your local VM's IP address into a browser window. You should be presented with the 'Hello World' message, along with the hostname of the container.

![](img/python-helloworld.png)

Stop and remove the `my-python` container

```
$ sudo docker rm $(docker stop my-python)
```

As the Docker image was built from the Dockerfile that used an environment variable, we can customise this when running the container.

For example, use the `NAME` variable to post a different message

```
$ sudo docker run -d --name my-python -p 8080:8080 -e NAME='Everybody' my-python:v1
```

Refresh the browser, the response should have changed.

We can also create a new version of this image by editing the source code and rebuilding the image.

Edit the `app.py` file and change the output message to something of your choice.

Rebuild the image

```
$ sudo docker build -t my-python:v2 .
```

Create a new container using the new image, this time using port 81 on the host

```
$ sudo docker run -d --name my-python2 -p 81:8080 my-python:v2
```

In the browser point to the new container at `localhost:81` to see your new message.

Clean up both containers using `docker stop <container-name>` and `docker rm <container-name>`.

Lab complete. Please move on to [2 Tier Monitoring Application](2-tier-monitoring-application-ex-5.md)