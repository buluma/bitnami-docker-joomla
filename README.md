[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-joomla/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-joomla/tree/master)
[![Slack](https://img.shields.io/badge/slack-join%20chat%20%E2%86%92-e01563.svg)](http://slack.oss.bitnami.com)

# What is Joomla?

> Joomla is a PHP content management system (CMS) for publishing web content. It includes features such as page caching, RSS feeds, printable versions of pages, news flashes, blogs, search, and support for non-english languages.

https://www.joomla.org/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-joomla/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run Joomla with a Database Container

Running Joomla with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run Joomla. You can use the following `docker-compose.yml` template:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - 'mariadb_data:/bitnami'
  joomla:
    image: 'bitnami/joomla:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'joomla_data:/bitnami'
    depends_on:
      - mariadb

volumes:
  mariadb_data:
    driver: local
  joomla_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create joomla-tier
  ```

2. Start a MariaDB database in the network generated:

  ```bash
  $ docker run -d --name mariadb -e ALLOW_EMPTY_PASSWORD=yes --net joomla-tier bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to Joomla to resolve the host

3. Run the Joomla container:

  ```bash
  $ docker run -d -p 80:80 -p 443:443 --name joomla --net joomla-tier bitnami/joomla
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define docker volumes namely `mariadb_data` and `joomla_data`. The Joomla application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - '/path/to/mariadb_persistence:/bitnami'
  joomla:
    image: 'bitnami/joomla:latest'
    depends_on:
      - mariadb
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/joomla-persistence:/bitnami'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create joomla-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb -e ALLOW_EMPTY_PASSWORD=yes \
    --net joomla-tier \
    --volume /path/to/mariadb-persistence:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to Joomla to resolve the host

3. Create the Joomla container with host volumes:

  ```bash
  $ docker run -d --name joomla -p 80:80 -p 443:443 \
    --net joomla-tier \
    --volume /path/to/joomla-persistence:/bitnami \
    bitnami/joomla:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and Joomla, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Joomla container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/joomla:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop joomla`
 * For manual execution: `$ docker stop joomla`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/joomla-persistence /path/to/joomla-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v joomla`
 * For manual execution: `$ docker rm -v joomla`

5. Run the new image

 * For docker-compose: `$ docker-compose start joomla`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name joomla bitnami/joomla:latest`

6. You need to complete the upgrade from the browser to get latest Joomla version.

# Configuration

## Environment variables

When you start the joomla image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
joomla:
  image: bitnami/joomla:latest
  ports:
    - 80:80
    - 443:443B
  environment:
    - JOOMLA_PASSWORD=my_password
```

 * For manual execution add a `-e` option with each variable and value:

  ```bash
  $ docker run -d --name joomla -p 80:80 -p 443:443 \
    -e JOOMLA_PASSWORD=my_password
    --net joomla-tier \
    --volume /path/to/joomla-persistence:/bitnami \
    bitnami/joomla:latest
  ```

Available variables:

 - `JOOMLA_USERNAME`: Joomla application username. Default: **user**
 - `JOOMLA_PASSWORD`: Joomla application password. Default: **bitnami**
 - `JOOMLA_EMAIL`: Joomla application email. Default: **user@example.com**
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**

### SMTP Configuration

To configure Joomla to send email using SMTP you can set the following environment variables:

 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_PROTOCOL`: SMTP protocol.

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  joomla:
    image: bitnami/joomla:latest
    ports:
      - 80:80
      - 443:443
    environment:
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
```

 * For manual execution:

  ```bash
  $ docker run -d --name joomla -p 80:80 -p 443:443 \
    -e SMTP_HOST=smtp.gmail.com \
    -e SMTP_PORT=587 \
    -e SMTP_USER=your_email@gmail.com \
    -e SMTP_PASSWORD=your_password
    --net joomla-tier \
    --volume /path/to/joomla-persistence:/bitnami \
    bitnami/joomla:latest
  ```

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-joomla/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-joomla/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-joomla/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright 2016-2018 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
