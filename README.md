# Gitlab CI/CD
A 0 to production tutorial using Gitlab for management of continuous integration and continuous deployment. In this tutorial our setup enviroment will be:
- GitLab `(Source base code)`
- Digital Ocean `(VM host)`
- Docker `(Containers)`

## Summary
1. [Create a droplet](#digital-ocean)
2. [Configure your droplet](#config)
3. [Create a gitlab runner](#runner)
4. [Create a docker image](#docker)
5. [Gitlab CI](#ci)
6. [Deploying your application](#deploy)

<div id='digital-ocean'/>

### Create a droplet

After made a account on [Digital Ocean website](https://www.digitalocean.com/), at the initial page you will click to create a `new project` on Digital Ocean:

![Projects page](img/project.png)

Now you have a project on Digital Ocean, that basically serves for centralize one, or more, droplets in a same group. You should click on `Create a droplet` to choose a type of droplet and your size. Because we are focused  on create a enviroment using Docker for deploy automatization, we'll also choose a droplet that fits with that environment.

![Docker droplet](img/docker-droplet.png)

Choosing a docker droplet, the `Digital Ocean` will give us a VM that already contains docker installed and also a firewall configuration that allows us to access the standard docker port remotely.

#### SSH Access





