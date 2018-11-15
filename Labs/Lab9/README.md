# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 9

In this lab, we'll explore using Docker to create a deployable image of our web server.

## Table of Contents
- [Installing Docker](#install)
- [Creating a Basic Dockerfile](#dockerfile1)
- [Building and Running Docker Images](#dockerimage)
- [Using the Docker Repository](#registry)

<a name="install"></a>
## Installing Docker (2 marks)

First, we'll need to install Docker on our server. We'll begin by installing dependencies first:

```sudo yum install -y yum-utils device-mapper-persistent-data lvm2```

Then, we'll using the yum-config-manager to add the Docker repo:

```sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo```

Finally, we can install Docker, enable it, and start it:

```sudo yum install docker-ce```
```sudo systemctl start docker```
```sudo systemctl enable docker```

Test that Docker has install correctly by running the default Docker Hello World application:

```sudo docker run hello-world```

<a name="dockerfile1"></a>
## Creating a Basic Dockerfile (5 marks)

To get the hang of working with Docker, we'll create a basic dockerfile and use it to create an image of a very simple web page.

First, we'll need to get the proper base image for our containers. Use the command ```sudo docker pull centos``` to get the latest version of the ```centos``` base image.

Now we need to create the dockerfile. Navigate to your home folder (```~```) and run ```nano dockerfile```.

The first line in our dockerfile will define what base image we're using, in our case the ```centos``` image. Thus, the syntax is ```FROM centos```.

The next line will be our ```RUN``` statements. You'll use two ```RUN``` statements, one to install the Apache web server, and one to install PHP. To install PHP, you can just use the ```yum -y install php``` - you don't need to worry about getting a different version than what is packaged with CentOS. Remember that you'll need to use the ```-y``` flag to auto-confirm install dialogs.

Create another file within your home folder called ```lab9.php```. Within that file, copy and past the following lines:

	<?php
	echo "<h1>Hello COSC419! PHP</h1>";
	echo "Current Time: " . date('Y-m-d H:i:s');
	
Now, use the ```COPY``` command in your dockerfile to copy ```lab9.php``` to the Apache default document root (```var/www/html```) in your container, with the name ```index.php```.

Use the ```EXPOSE``` command to open port 80 so that we can interact with our web server.

Finally, add the following line at the bottom of your dockerfile:

```CMD apachectl -D FOREGROUND```

This will actually start Apache when your docker instance is run.

<a name="dockerimage"></a>
## Building and Running Docker Images (2 marks)

Now that you've created a dockerfile, we'll use it to build an image, and then run that image on our server.

To build your dockerfile, use the following command:

```sudo docker build -t <imageName> ~```

You may use any name for your image. If your dockerfile is configured correctly, you should see Docker follow through each command and then report that it has successfully built the image. You can double check this by running the following command:

```sudo docker images```

This will list all images on your machine. In addition the the hello-world and centos images, you should see the image that you have just created.

Next, we'll run our Docker instance. Use ```sudo docker run```. Make sure that you use the ```-d``` and ```-p``` flags to detach the Docker instance and set the exterior port to ```8888```, and specify the correct image name. Afterwards, visit your website at ```http://<yourIp>:8888```. If your Docker instance is working correctly, you should see a page that looks like the following:

<img src="https://i.imgur.com/PIuSI3D.png">

<a name="registry"></a>
## Using the Docker Repository (1 marks)

First, go to the Docker <a href="https://hub.docker.com">website</a>, create an account, and verify your email address. Then, in your terminal, call ```sudo docker login```, and enter your account username and password.

Once you've done that, use the ```sudo docker build``` command again, but specify an image name that fits the following format:

```<dockerusername>/cosc419lab9:new```

Note that it must all be lowercase - use of capital letters such as in camel-case names will result in Docker throwing an error.

Once you've done that, you can then push your new Docker image to the online repository with the following command:

```sudo docker push <dockerusername>/cosc419lab9:new```

When the command is finished running, check the repository website and verify that your image is now available online. You can double-check this by running the following command:

```sudo docker pull <dockerusername>/dockertest:new```

This will attempt to pull the image, but since we already have an identical copy, will instead return that the image is up to date.

When you're done, email me your name, student number, and a link to your Docker repo at ```mattfritter@gmail.com```.
