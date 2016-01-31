<!--[metadata]>
+++
title = "Getting started with Docker Swarm"
description = "Introduction to Swarm commands and concepts"
keywords = ["docker, swarm, clustering, discovery, examples"]
[menu.main]
parent="workw_swarm"
+++
<![end-metadata]-->

# Evaluate Swarm on Mac or Windows

This getting started example shows you how to create a Docker Swarm cluster. The tutorial is intended for users who are running Mac or Windows and want to evaluate Swarm locally on their system.

When you finish, you'll have a Docker Swarm up and running in VirtualBox on your
local Mac or Windows computer. You can use this Swarm as personal development
sandbox to evaluate Swarm.

To use Docker Swarm on Linux, see [Install Swarm for production](https://docs.docker.com/swarm/install-manual/).

## Install Docker Toolbox

Download and install [https://www.docker.com/docker-toolbox](Docker Toolbox).

The toolbox installs a handful of tools on your local Windows or Mac OS X computer. In this exercise, you use three of those tools:

 - Docker Machine: To deploy virtual machines that run Docker Engine.
 - VirtualBox: To host the virtual machines deployed from Docker Machine.
 - Docker Client: To connect from your local computer to the Docker Engines on the VMs and issue docker commands to create the Swarm.

The following sections provide more information on each of these tools. The rest of the document uses the abbreviation, VM, for virtual machine.

## Create three VMs running Docker Engine

This evaluation is intended for users on Mac OSX and Windows.  Neither of these
operating systems run Docker Engine natively.  For this reason, you'll use
Docker Machine to provision three VMs in Virtualbox.  The VMs will run a
small-footprint Linux OS, `boot2docker.iso`, that can host and run the Docker
Engine software.  Once you create three VMs, you can use the Swarm interactive
container to cluster and run Swarm on the three virtual Linux hosts.

1. Open a terminal on your computer. Use Docker Machine to list any VMs in VirtualBox.

		$ docker-machine ls
		NAME         ACTIVE   DRIVER       STATE     URL                         SWARM
		default    *        virtualbox   Running   tcp://192.168.99.100:2376   

2. Optional: To conserve system resources, stop any virtual machines you are not using. For example, to stop the VM named `default`, enter:

		$ docker-machine stop default

3. Create and run a VM named `manager`.  

    $ docker-machine create -d virtualbox manager
    Running pre-create checks...
    Creating machine...
    (manager) Copying /Users/mary/.docker/machine/cache/boot2docker.iso to /Users/mary/.docker/machine/machines/manager/boot2docker.iso...
    (manager) Creating VirtualBox VM...
    (manager) Creating SSH key...
    (manager) Starting the VM...
    (manager) Waiting for an IP...
    Waiting for machine to be running, this may take a few minutes...
    Machine is running, waiting for SSH to be available...
    Detecting operating system of created instance...
    Detecting the provisioner...
    Provisioning with boot2docker...
    Copying certs to the local machine directory...
    Copying certs to the remote machine...
    Setting Docker configuration on the remote daemon...
    Checking connection to Docker...
    Docker is up and running!
    To see how to connect Docker to this machine, run: docker-machine env manager

4. Create and run a VM named `agent1`.  

		$ docker-machine create -d virtualbox agent1

5. Create and run a VM named `agent2`.  

		$ docker-machine create -d virtualbox agent2

6. List all the machines you created.

    $ docker-machine ls


## Create a Swarm discovery token

Swarm networks rely on a discovery service to maintain a list of nodes in a
Swarm cluster. A discovery service associates a token with instances of the
Docker Engine daemon running on each node. In this step, you generate the
discovery service token.

There are several types of discovery backends. For this evaluation, you'll use
Docker Hub as a hosted discovery service. You should only use Docker Hub as a
discovery service with Swarm if you are evaluating or testing. Docker Hub
service may fail in which case it can cause your Swarm cluster operations that
rely on it to fail also.

**Note**: If you experience problems with Swarm while using Docker Hub as discovery
backend, check the Docker Hub [status page](http://status.docker.com/) as it
could be the source of your problem.


1. Connect the Docker Client on your computer to the Docker Engine running on `manager`.

        $ eval $(docker-machine env manager)

    The client will send the `docker` commands in the following steps to the Docker Engine on on `manager`.

2.  Create a unique id for the Swarm cluster.

        $ docker run --rm swarm create
        Unable to find image 'swarm:latest' locally
        latest: Pulling from library/swarm
        d681c900c6e3: Pull complete
        188de6f24f3f: Pull complete
        90b2ffb8d338: Pull complete
        237af4efea94: Pull complete
        3b3fc6f62107: Pull complete
        7e6c9135b308: Pull complete
        986340ab62f0: Pull complete
        a9975e2cc0a3: Pull complete
        Digest: sha256:c21fd414b0488637b1f05f13a59b032a3f9da5d818d31da1a4ca98a84c0c781b
        Status: Downloaded newer image for swarm:latest
        dd0963dab9033e37a3f43cfa33d0bd5b


    The `docker run` command gets the latest `swarm` image and runs it as a container. The `create` argument makes the Swarm container connect to the Docker Hub discovery service and get a unique Swarm ID, also known as a "discovery token". The value returned by the command is the `dd0963dab9033e37a3f43cfa33d0bd5b` value at the end of the output.





    The discovery service keeps unused tokens for approximately one week.

3. Copy the discovery token from the last line of `swarm create` output above.

4. Store the token to a safe place as you'll use it later.

## Create three Swarm nodes

1. Get the IP addresses of you Swarm manager.

    As you are using `docker-machine` to manage your nodes, you can use the `ip`
    command.

      $ docker-machine ip manager
      192.168.99.101  

    In this example, the IP is `192.168.99.101`.

2. Start the Swarm manager on `manager` node.

    The command to run the manager node has this syntax:

        docker run -t -p <your_selected_port>:2375 -t swarm manage token://<cluster_id> >

    For example:

        docker run -t -p 2370:2375 -t swarm manage token://f6dc2febe7d321b987d30228e8c7a21b

3. Connect Docker Client to `agent1`.

        eval $(docker-machine env agent1)

4. Use the following syntax to run a Swarm container as an agent on `agent1`.

        docker run -d swarm join --addr=<node_ip:2375> token://<cluster_id>

    For example:

        docker run -d swarm join --addr=192.168.99.101:2375 token://f6dc2febe7d321b987d30228e8c7a21b

5. Connect Docker Client to `agent2`.

        eval $(docker-machine env agent2)

6.  Run a Swarm container as an agent on `agent2`.

        docker run -d swarm join --addr=192.168.99.102:2376 token://f6dc2febe7d321b987d30228e8c7a21b

## Manage your Swarm

1. Connect the Docker Client to the Swarm.

        $ eval $(docker-machine env swarm)

    Because Docker Swarm uses the standard Docker API, you can connect to it using  Docker Client and other tools such as Docker Compose, Dokku, Jenkins, and Krane, among others.  

2. Get information about the Swarm.

        $ docker info

    As you can see, the output displays information about the two agent nodes and the one master node in the Swarm.

3. Check the images currently running on your Swarm.

        $ docker ps

4. Run a container on the Swarm.

        $ docker run hello-world
        Hello from Docker.
        .
        .
        .

5. Use the `docker ps` command to find out which node the container ran on.

        $ docker ps  -a
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
        0b0628349187        hello-world         "/hello"                 20 minutes ago      Exited (0) 20 minutes ago                       agent1
        .
        .
        .

    In this case, the Swarm ran 'hello-world' on the 'swarm1'.

    By default, Docker Swarm uses the "spread" strategy to choose which node runs a container. When you run multiple containers, the spread strategy assigns each container to the node with the fewest containers.

## Where to go next

At this point, you've done the following:
 - Created a Swarm discovery token.
 - Created Swarm nodes using Docker Machine.
 - Managed a Swarm and run containers on it.
 - Learned Swarm-related concepts and terminology.

However, Docker Swarm has many other aspects and capabilities.
For more information, visit [the Swarm landing page](https://www.docker.com/docker-swarm) or read the [Swarm documentation](https://docs.docker.com/swarm/).

<[The Run reference](https://docs.docker.com/engine/reference/run/)>
<Link to information about High-availability Swarm>
<Link to using TLS>
<Link to using different Strategies>
