# php72-devbox

This repository contains the files needed to create both a development and integration environment for Magento using docker containers.

# Intro

This image is designed to serve as a development environment for upgrading the Magento2 stack to php7.2. The Docker image brings up three containers:

**Database:** mySQL 5.6

**Service:** Redis, Varnish, Elasticsearch and RabbitMQ

**Web:** PHP 7.2.1, Apache2

# Instructions

Prerequesits : Docker Community Edition

**Step 1**: Open a terminal window and clone the php72-devbox repository on your local system.
The name you choose as the folder name will be used later so make sure you name it
something descriptive. When the containers get created in step 3 Docker uses the folder
name as the beginning part of the container name followed by the service name and
instance. ex myfolder_web_1

```
mkdir myfolder
cd myfolder
git clone https://github.com/giabmf11/php72-devbox.git
```

**Step 2**: Run the following command to build the web image.

```
docker-compose build
```

**Step 3**: Run the following command to create the containers by executing this command.

```
docker-compose up -d
```

**Step 4**: Run the following command to find the port docker is exposing for the web container's port 80. The format
will be 0.0.0.0:[port]. Remember the port because you will need it later. ex 32909

```
docker-compose port web 80
```

**Step 5**: Run the following command to see the new containers and their details.

```
docker ps
```

**Step 6**: Find the web container and copy it's name. The web container name will be in the following format: [FolderName]-web-1.

**Step 7**: Copy your Magento source files into the web container. Make sure you change the following commands to copy the source code into the web container name you copied in step 5 and the Magento code base you are modifying.

Here is the format:

```
docker cp ~/[MAGENTO_REPO_LOCATION] [CONTAINER_FROM_STEP_5]:/var/www/magento2ce
```

Example:

```
docker cp ~/git/Magento dockertest_web_1:/var/www/magento2ce
```

**Step 8**: Run the relink.sh script in the web container to set up the installation.

```
docker-compose exec --user=magento2 web relink.sh
```

**Step 9**: Using the port from step 4 run the install.sh script in the web container.

Here is the format:

```
docker-compose exec --user=magento2 web install.sh [PORT_FROM_STEP_4]
```

Example:

```
docker-compose exec --user=magento2 web install.sh 32909
```

**Step 10**: Navigate to your instance by going to http://localhost:[port from step 4]. ex http://localhost:32909.


# Comments, questions, bug reports, contributions?

Please use GitHub issue/pull request features to ask questions, reports bugs or provide contributions.

# What hosts are supported?

* 64-bit Windows 10 Pro, Enterprise and Education (1511 November update, Build 10586 or later)
* Mac OS 10.10.3 "Yosemite" or later

# My site doesn't work after restart

Docker assigns a random free port to the container on restart. Run the m2devbox-reset script to reinitialize containers to use the proper port.

# Why do you use a script to generate the container files?

Using a script helps us provide a unique name for the containers, which allows you to run multiple sets of containers on one host. The script wraps the commands and also enables containers to run without conflict.
In the future, keys will be integrated so there will be no need to enter them.

# Why are Apache and PHP in the same container? Docker best practices suggest a container for each component...

Our goal was to create an easy to use development environment, not a production environment. The containers in Magento DevBox should not be used as a model for production use.

Additionally, when we tried separate containers, we encountered stability issues, especially when running multiple sets of containers. Docker is a quickly evolving product but it's not yet sufficiently stable. We are eager to revisit this approach and split the container in components, with option for nginx instead of Apache.

# What do you use for file sharing? What is this syncing process?

* MacOS: The shared filesystem is not performant enough. Magento worked too slowly.

That is why we decided to use Unison (a file synchronization tool) to keep the local (inside Docker) web root directory in sync with files in the shared directory. It works quite well, although the initial synchronization takes a few minutes.

We decided not to put Unison on the host machine (which would allow us not to use shared directories at all) to avoid adding dependencies. The MacOS shared filesystem supports notification events, so it can be used directly.

* Windows: The shared file system does not support notification events.

For this reason, we require a locally installed Unison file synchronization tool, so it can pick up file changes.

# The initial startup is slow - why?

Our goal was to provide maximum flexibility. The container supports multiple methods of installing Magento (new installation, from existing local Magento code, or from Magento Enterprise Cloud Edition) and multiple editions (CE, EE, B2B).

For this reason, we do not include Magento files in the Docker image, but instead use Composer to download the files on first start. This is a slower approach than other images you might find on Docker hub; however, it allows us to configure Magento to suit your needs.

We are considering embedding some of the files into the container for a faster start.

# Why do you include Redis, Varnish, Elasticsearch and RabbitMQ?

We think that you should develop as much as possible in an environment resembling a production configuration. In the majority of cases, a production configuration includes those components (Elasticsearch and RabbitMQ for Magento EE deployments).

This approach allows you to catch early issues related to incompatibility instead of finding those issues at the last moment. During Magento DevBox installation, use *Advanced Options* to choose which of these components to install.

# Warmup

By default, to save installation time, no warmup is performed. If you would like to use the container for demo purposes, you can enable warmup. More information is available in documention http://devdocs.magento.com/guides/v2.1/install-gde/docker/docker-commands.html.

# Cron

By default, to save batteries/energy, cron is disabled. Our experience shows, that running cron in container results in very quick draining of laptop batteries. To enable cron, you can follow the instructions in the documentation http://devdocs.magento.com/guides/v2.1/install-gde/docker/docker-commands.html.

