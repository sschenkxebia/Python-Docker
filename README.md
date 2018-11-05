# Python-Vscode

## 1. Introduction

This repository is a condensed walkthrough of the official Docker [Getting Started Guide](https://docs.docker.com/get-started/), using python in vscode, with the docker and python extensions.

## 2. Docker

Develop, deploy, and run applications with containers.

1. **Image:** a blueprint for a container
1. **Container:** a unit of software (instance of an image) that packages all its dependencies so that it run smoothly and reliably
1. **Swarm:** a cluster of nodes (running Docker)
1. **Service:** a collection of containers running the same image
1. **Stack:** multiple services, possibly sharing dependencies, to be scaled and orchestrated together

For example: an application, or a stack, may contain a database service, and a web-app service, described by their respective images, each having three containers (6 in total), running distributed on a swarm.

## 2.1 Build Image

Verify that the Dockerfile exists

    $ ls
    Dockerfile		app.py			requirements.txt

Build the image from the dockerfile

    docker build -t menziess/python-vscode:latest .

Show the image that has been built

    $ docker image ls

    REPOSITORY             TAG                 IMAGE ID
    menziess/python-vscode latest              326387cea398

Run the app in detached mode

    docker run -p 4000:80 menziess/python-vscode

Show the running containers

    $ docker container ls
    CONTAINER ID        IMAGE                  COMMAND             CREATED
    1fa4ab2cf395        menziess/python-vscode "python app.py"     28 seconds ago

Stop the running container

    docker container stop 1fa4ab2cf395

Push container

    docker login
    docker push menziess/python-vscode

## 2.2 Scale Service (Multiple Containers - Single Node)

Verify that the `docker-compose.yml` file exists

    $ ls
    Dockerfile		app.py			requirements.txt      docker-compose.yml

We initialize a swarm (of one node, our local computer)

    docker swarm init

Then we deploy our service

    docker stack deploy -c docker-compose.yml getstartedlab

And we scale it by increasing the number of replicas in the yml file, and simply run the previous command again

    docker stack deploy -c docker-compose.yml getstartedlab

We take down the app, and leave the swarm

    docker stack rm getstartedlab
    docker swarm leave --force

## 2.3 Distributed Swarm (Containers Across Cluster - Multiple Nodes)

Start two VM's using virtualbox and docker-machine

    docker-machine create --driver virtualbox myvm1
    docker-machine create --driver virtualbox myvm2

Initialize the first VM as a swarm manager, as we did in 2.2

    docker-machine ls
    docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"

Copy the command that is show in terminal, and ssh into the second VM, then paste the command to add it as a worker to the swarm

    docker-machine ssh myvm2 "<the command that is shown>"

Show all the nodes in the swarm

    docker-machine ssh myvm1 "docker node ls"

Now you can walk through 2.2 again, and deploy the stack, but on the distributed swarm this time. If you do this, run `docker-machine ls` to reveal the VM ip addresses to access the application in the browser.

Finally, we can leave the swarm from within each VM, remove the stack

    docker-machine ssh myvm2 "docker swarm leave"
    docker-machine ssh myvm1 "docker swarm leave --force"

    docker stack rm getstartedlab
    docker-machine stop myvm1
    docker-machine stop myvm2

## 2.4 Stack Services (Adding Database)

We will expand our `docker-compose.yml` file by adding more services. We will add a docker visualizer and a redis database. The database will require a volume that is stored on the swarm manager called `/data`, let's make that folder and redeploy

    docker-machine ssh myvm1 "mkdir ./data"
    docker stack deploy -c docker-compose.yml getstartedlab

Our application should now display the number of visits.

## 3. Development

A container can be run with a shared volume, so that code changes are immediately visible, and the container is deleted after use

    docker run -p 80:80 --rm -v "$PWD:/app" menziess/python-vscode

