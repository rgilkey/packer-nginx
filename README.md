# Overview

This repo provides an example of how to use [packer](https://packer.io/) to build a generic [nginx](http://nginx.com/) server from either a [CentOS](http://centos.org/) or [Ubuntu](https://ubuntu.com/) container image.  The basic workflow is as follows:

1. Packer pulls the desired container image
2. It uses a shell provisioner for the Ubuntu image to ensure that `python` is installed.  CentOS bundles python since there are a few dependencies on python built into the distribution.
3. Runs an [Ansible](https://www.ansible.com/) playbook against the container to configure the system.  The `ansible-remote` method is utilized to prevent any remnants of Ansible from being installed on the container.
4. Tags and pushes the container to a [Docker Hub](http://hub.docker.com/) account

# Prerequisites

There are a few prerequisites needed to make it all go:

1. Packer - https://packer.io/docs/install/index.html
2. Ansible 2.7+ - https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
3. Docker - https://docs.docker.com/install/
4. A Docker Hub account - https://hub.docker.com/signup

# Supported variables

The packer template supports a few user supplied variables:

* `container_image` (required) - should be either `centos` or `ubuntu` to pull the corresponding image from Docker Hub
* `container_tag` (optional) - defaults to `latest`, a more fun value like `git rev-parse --short HEAD` could be used to facilitate promotions
* `docker_hub_user` (required) - the Docker Hub user account to push containers to

# Building

The image can be built and shipped via the following command:

```sh
packer build -var container_image=<centos|ubuntu> -var docker_hub_user=<docker_hub_username> template.json
```

The build process assumes that the docker daemon is already logged in to the Docker Hub account.  This process is separated from the build step so that:

1. Credentials can be isolated from the stack repository
2. Credentials could be passed in to the build process in a pipeline and handled in a more secure fashion

# Running the container

Once the container is built, it could be run as shown below.  This repositry includes a sample `html` folder containing an `index.html` file for demonstration purposes.

```sh
docker run --name nginx-test -v <path_to_repo_checkout>/html:/html -p 8888:80 -d <docker_hub_username>/<centos|ubuntu>-nginx:latest
```

Once the container is running it can be viewed in a web browser at http://127.0.0.1:8888/.

## Container logs

The nginx daemon writes its logs to its stderr stream so that they can be accessed via the docker daemon or the container platform's designated logging system.
