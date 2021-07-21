# ansible-in-docker
# Ansible with Docker

This project is a created for study purposes.

The objective is to understand Ansible and see it in action, without using any cloud support, but instead, plain Docker containers as target machines.

## Architecture

This project is built using Docker compose, which allows us to start the whole system, as we would have running against some cloud environment, like Amazon.

Compose contains two services:

- **ansible**: the container with Ansible installed and ready to use. This container will play the role as our main machine, in which the Ansible commands will be dispatched.

- **server**: a plain container, with SSHD installed and configured. This container will play the role as our cloud machines.

## Setup

The Ansible container has mapped some extra files, present on this repository:

- **/ssh/config**: this file contains some specific client SSH configuration, necessary in order to the Ansible container to connect to the server containers

- **/playbooks**: directory containing our custom Ansible playbooks, responsible to configure the server machines

- **/ansible/hosts**: contains the hosts that will be reached by `ansible-playbook`. Today, it's hardcoded to 3 machines, using the container names generated by Docker compose (information about how to start all the three instances can be found on the `Running` section).

## Running

### Start the base system

First, we start the system with the Ansible container + 1 server instance:

```bash
$ docker-compose run ansible bash
```

This command will build the server image based on the `Dockerfile` contained on this repository. This image basically generates a clean container with SSHD installed and configured to receive connections. After the image is created, both containers are started and a console to Ansible container is open.

### Start the servers

Now, we need to start the other 2 server machines. So, in a different terminal, run:

```bash
$ docker-compose scale server=3
```

You can check now, using `docker ps`, that we've four containers running: 1 Ansible container + 3 server containers, named:
- ansibledocker_server_1
- ansibledocker_server_2
- ansibledocker_server_3

### Test servers connection

Back to the Ansible terminal, you can test that all server containers are reachable running:

```bash
$ ansible all -m ping
```

### Install extra Ansible roles

Before running the Ansible playbook, we need to download some extra roles, used in our playbook:

- **ANXS.git**: used to install the Git client on the server. Git will be used to download the NodeJS test project from Github.

- **geerlingguy.git**: Ansible Git interface, used to download the project from Github and store inside the container

- **geerlingguy.nodejs**: install and setup NodeJS on the server

So, to download the roles:

```bash
$ ansible-galaxy install ANXS.git geerlingguy.git geerlingguy.nodejs
```

### Running the playbook

Now, we can play the Ansible playbook:

```bash
$ ansible-playbook /root/playbooks/setup.yml
```

This process can take some time, since Ansible will install all the necessary dependencies and configure the services. You can follow the Ansible logs to check the magic being done.

### Checking

After the servers configured, you can now check that all the 3 servers are configured and with our hello world project running.

So:

```bash
$ curl -XGET ansibledocker_server_1:5000
$ curl -XGET ansibledocker_server_2:5000
$ curl -XGET ansibledocker_server_3:5000
```

All the three requests above should return a `Hello World` message, meaning that the Node server is up and running.

## Success

You now have fully setup 3 Docker containers using Ansible and running your project. :)

## Observations:

- The objective of this project is not teach how to provision Docker containers using Ansible. As stated by Michael DeHaan, creator of Ansible, Docker containers typically have a single responsibility and, thus, much less configuration. So, the overhead of having a complete Ansible configuration to provision them are unnecessary.

- This project uses Docker for convenience. This way, we don't have to bother about starting Amazon servers only for testing purposes. However, since the containers used here behave exactly like a plain machine with SSHD installed, the very same setup should work on any cloud or bare-metal architecture.
