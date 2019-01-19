[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)


# Gitlab CI/CD
A **zero to production** tutorial using Gitlab for CI/CD management. In this tutorial the setup enviroment will be:

- GitLab `(Source base code)`
- Digital Ocean `(Infrastructure)`
- Docker `(Containers)`

## Summary
1. [Create a droplet](#digital-ocean)
    1. [Create a public key](#ssh)
2. [Configure your droplet](#config)
3. [Create a gitlab runner](#runner)
4. [Create a docker image](#docker)
5. [Gitlab CI](#ci)
    1. [Enviroment Variables](#ci-ssh)
    2. [Create a gitlab-ci.yml](#ci-yml)
6. [Deploying your application](#deploy)

<div id='digital-ocean'/>

### Create a droplet

After made a account on [Digital Ocean website](https://www.digitalocean.com/), at the initial page you will click to create a `new project` on Digital Ocean:

![Projects page](img/project.png)

Now you have a project on Digital Ocean, that basically serves for centralize one, or more, droplets in a same group. Click on `Create a droplet` to choose a type of droplet and your size. Because we are focused  on create a enviroment using Docker for deploy automatization, we'll also choose a droplet that fits with that environment.

![Docker droplet](img/docker-droplet.png)

Choosing a docker droplet, the `Digital Ocean` will give us a VM that already contains docker installed and also a firewall configuration that allows us to access the standard docker port remotely.

### Note

> **If you intent to use this tutorial for deploy a Spring Boot Application, it's better to choose at least the U$10/mo machine, because the 1GB RAM isn't enough to build the project, you'll face memory overflow issues.**

<div id='ssh'/>

#### SSH Access

Add your public ssh key to `Digital Ocean`. You need to create a ssh from the machine that you want to access remotely your droplet. 

![SSH](img/ssh-key.png)

If you haven't already have a public ssh key, following this steps:

```
$ ssh-keygen -o
Generating public/private rsa key pair.
Enter file in which to save the key (/home/you/.ssh/id_rsa):
Created directory '/home/you/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/you/.ssh/id_rsa.
Your public key has been saved in /home/you/.ssh/id_rsa.pub.
```

And get your public ssh key: 

```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAasd24cZ/124zcA...you@you-pc
```
Copy and paste that on `Digital Ocean`, and that's it.

And after all, click on `Create`. 

<div id='config'/>

### Configure your droplet
After the creation of your droplet, you will able to see some information about the droplet, and the most important : `IP Address`

![IP](img/droplet.png)

Now, open a terminal and type

```
$ ssh root@your.ip.address
```

![Terminal](img/terminal.png)

### Note

> **If you can't connect remotely with some error like Connection Refused, maybe you should use the Digital Ocean Console Launcher for add manually your public ssh key on this  folder: ~/.ssh/authorized_keys**

Checking your firewall, you will see what ports are available to remote access. On the initial configuration you only have `SSH and Docker ports.`

```
$ ufw status
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
2375/tcp                   ALLOW       Anywhere
2376/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
2375/tcp (v6)              ALLOW       Anywhere (v6)        
2376/tcp (v6)              ALLOW       Anywhere (v6)
```

### Note

> **After you start a deploy your applications on this droplet, you should change this firewall configuration or also configure a proxy for remote access to your applications.**

<div id='runner'/>

### Create a gitlab runner
#### What is it

GitLab Runner is the open source project that is used to run your jobs and send the results back to GitLab. It is used in conjunction with GitLab CI, the open-source continuous integration service included with GitLab that coordinates the jobs.

#### Install
Simply download one of the binaries for your system:

```
# Linux x86-64
 $ sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

 # Linux x86
 $ sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-386

 # Linux arm
 $ sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm
```

Give it permissions to execute:

```
$ sudo chmod +x /usr/local/bin/gitlab-runner
```

Install and run as service:

```
$ sudo gitlab-runner install
$ sudo gitlab-runner start
```

#### Register a runner
Go back on your Gitlab project, on the lateral menu

`Settings > CI/CD > Runners (Expand)`

![Gitlab](img/gitlab.png)

First of all, click on `Disable Shared Runners`

Then go back to your droplet terminal and:

```
$ sudo gitlab-runner register
```

Enter your GitLab instance URL:

```
 Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
 https://gitlab.com
```

Enter the token you obtained to register the Runner:

```
Please enter the gitlab-ci token for this runner
xxxxxx
```

Enter the [tags associated with the Runner](https://docs.gitlab.com/ee/ci/runners/#using-tags), you can change this later in GitLab’s UI:

```
 Please enter the gitlab-ci tags for this runner (comma separated):
 build,test,prod,...
```

Enter the Runner executor:

```
 Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
 docker
```

If you chose Docker as your executor, you’ll be asked for the default image to be used for projects that do not define one in `.gitlab-ci.yml`:

```
 Please enter the Docker image (eg. ruby:2.1):
 alpine:latest
```

And that's it, if you refresh your gitlab runner's page on your repository, you will see:

![Runners info](img/runners.png)

<div id='docker'/>

### Create a docker image

Before we start to build our `.gitlab-ci.yml`, you should create your `Dockerfile` to create a docker image.

#### What's docker

Docker is a computer program that performs operating-system-level virtualization, also known as "containerization".

#### Dockerfile

Here we have a example of a Dockerfile for a Spring Boot Application, but you can create a image that will fit with your application, but remender `your runner should have the same base image.`

```Dockerfile
FROM openjdk:10.0.1-slim
VOLUME /tmp
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

### Note

> **You can see the official docker documentation to help you to create your own https://docs.docker.com/engine/reference/builder/**

<div id='ci'/>

### Gitlab CI

<div id='ci-ssh'/>

#### Enviroment Variables
Before we create the `.gitlab-ci.yml`, you need to generate a `private ssh key` on your **droplet**. It's simple, it's the same steps that we did before, but now in your droplet.

```
$ ssh-keygen -o
Generating public/private rsa key pair.
Enter file in which to save the key (/home/you/.ssh/id_rsa):
Created directory '/home/you/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/you/.ssh/id_rsa.
Your public key has been saved in /home/you/.ssh/id_rsa.pub.
```

And get your **private ssh key**: 

```
$ cat ~/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAnjPlxuZelQFBCTllfxErc6PNJ+U6f3zt03jf8/ckB3UonRmS ...
-----END RSA PRIVATE KEY-----
```

Copy, paste and create a variable called **SSH_PRIVATE_KEY** in `Gitlab Enviroment Variables`

![Gitlab SSH](img/gitlab-ssh.png)

That will allow us to send the files and execute commands inside of `.gitlab-ci.yml.`

<div id='ci-yml'/>

#### Create a gitlab-ci.yml

<div id='deploy'/>

### Deploying your application


### License
Apache License. [Click here for more information.](LICENSE)