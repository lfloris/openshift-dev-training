# Exercise 1 - Deploying stateless container

In this lab, you'll create a simple Apache container and connect to it from your local machine.
​
Create the container from the `httpd:2.4` image. This command also maps the local machine port 8080 to the container port 80.
​
```
$ sudo docker run -dit --name httpd-lab -p 8080:80 httpd:2.4
```

Docker will automatically download the image and create a container from it. Test the connection using `curl`, or from the browser go to `localhost:8080`
​
```
$ curl http://localhost:8080
```

You can create a terminal session to the container by using `docker exec`
​
```
$ sudo docker exec -it httpd-lab /bin/bash
```
​
Modify the `index.html` file and exit the container
​
```
$ echo "I've changed" > htdocs/index.html
$ exit
```
​
Test the connection again. You should see the new message.
```
$ curl http://localhost:8080
<html><body><h1>It works!</h1></body></html>
```
​
Because container storage is ephemeral, once the container is removed any changes you've made will be reverted back to the original configuration in the image. 

Stop the current container and remove it
​
```
$ sudo docker stop httpd-lab
httpd-lab
$ sudo docker rm httpd-lab
httpd-lab
```
​
Now deploy the Apache container again and you will see that the changes you made earlier are no longer present.

Lab complete. Please move on to [Creating a Stateful Container](stateful-container-ex-2.md)