# Logging in to the OpenShift Environment

Open the Student X VM window and log in with the following credentials

User: ibmdemo

Password: passw0rd

# Using the OpenShift Console

In your Student-X VM, open the Firefox browser. The link to the OpenShift environment is bookmarked.

![](img/ocp-bookmark.png)

From the log in screen, select `htpasswd` as the log in.

Enter the user name and password provided to you by the instructor. The username is typically `userXX` and the password will be supplied to you.

![](img/login-screen.png)

Select 'Log In'. This should take you to the OpenShift Project Dashboard

![](img/project-dashboard.png)

# Using the oc CLI

In your Student X VM, open a new Terminal window.

To log in, use `oc login` with the OpenShift API Server endpoint. You'll be prompted to enter your user name and password. Once successfully authenticated, you'll receive a Login successful messsage.

For this lab, the  OpenShift API Server endpoint is https://api.demo.ibmdte.net:6443.

```
$ oc login https://api.demo.ibmdte.net:6443
Authentication required for https://api.demo.ibmdte.net:6443 (openshift)
Username: user99
Password: 
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>
```



