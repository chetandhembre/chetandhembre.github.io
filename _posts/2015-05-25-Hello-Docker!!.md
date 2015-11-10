---
layout: post
title: "Hello Docker !!"
date: 2015-05-25 22:35:21 +0530
comments: true
categories:
- docker
- nginx
- image
---

Docker !! , Docker has been buzz world in technology world lately, and many people
are jumping into it. But there are so many what/why/how questions about docker is
present in newbie's mind.

First I wont go in depth details about in docker in this post.

Lets get answers for these questions step by step.

##What is Docker ?

here is description for docker from http://www.docker.com

```
Docker is an open platform for developers and sysadmins to build, ship, and run
distributed applications

```
So first docker is open source platform which helps not only developer develop distributed
systems but also help sysadmins manage these systems in diverse environment.


This is containers for our application like we have containers in shipping industry.

##Ok But why we need Docker ?

If you look around, you will see your development team is using variety of OS
for developing application most probably they are using windows/OSX and we usually
have linux based system in production. Problem
with this is many technology which we uses for building this systems usually behave
differently is different OS. Many time you will see people complaining about
this code is working on my machine but not working on production  or my colleague's
machine.

But when you ``Dockerized`` your app you can use it on any OS. Thats really make
things simpler not only for developer but also sysadmin.

Also docker provide on hub where pre build Dockerized apps are present which are
very great to getting started with project.

##How is this different from Virtual Machines?

As you mentioned earlier that docker is container, next question pops into your
mind is `isn't it similar to Virtual Machines ?`. And this is valid question.

Here is explaination for this from http://www.docker.com

![Docker VS Virtual Machines](http://siliconangle.com/files/2014/08/Docker-vs-Virtualization.jpg)


From this it is clear that docker is more portable, light weight than VM.
which is great.

#How can we use Docker ?

So all you excited about docker ? Lets use them in hosting simple static website

I wont cover installation of docker here. There is great explaination [here](https://docs.docker.com/installation/#installation).

Lets do something simple but exciting. Lets run our simple Hello world static
website. What is better than `Nginx` to server static content.

Lets create directory named HelloDocker.

```
mkdir HelloDocker
cd HelloDocker
```

Create directory where we will have simple `index.html`. And our simple html

```

mkdir app
touch ./app/index.html
echo "<h1>Hellow Docker!!</h1>" > ./app/index.html

```
Docker require `Dockerfile` in root directory of you app to identity it is `Dockerized` app

```
touch Dockerfile

```
Add following code in your `Dockerfile`

```
FROM kyma/docker-nginx
ADD app/ /var/www
CMD 'nginx'

```
Let me explain code we added in Dockerfile.
As I mentioned earlier docker hub has pre build docker images for many applications.
So in first line we are imporing `nginx` image from hub using `FROM kyma/docker-nginx`.
This makes our docker container extendable which help us to make reusable docker images.

To host our website in nginx we have to move content of `app` folder to `/var/www` folder provided by
imported nginx docker image.

Lastly just run `nginx` to start our nginx.

You might be seeing lots of costume command in Dockerfile which are specific for docker.
You can learn about it [here](https://docs.docker.com/reference/commandline/cli/)

PS: remember code in Dockerfile is specific to our imported nginx image. If you change image you might require to change code.

So lets build our first docker image using running following command in root directory

```
docker build -t chetan/HelloDocker .

```

Here we are creating docker image of our app and tagged it as `chetan/HelloDocker`. And last `.` is for directory from which
we have to create docker image.

You can check your image by runing following command

```
docker image

```
Lets run this our simple docker app by following command

```
docker run -p 8080:80 -d chetan/HelloDocker

```

Running your image with -d runs the container in detached mode, leaving the container running in the background.
The -p flag redirects a public port(8080) to a private port(80) in the container.

To check our website is working run following command

```
curl localhost:8080

```

Yeah!!! we run our first docker app. Isn't it simple ?

Happy Dockering !!!

PS: If you are using `boot2docker` to run docker then run following command to visit our website

```
curl $(boot2docker ip):8080
```
