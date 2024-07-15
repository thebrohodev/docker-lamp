# ![Docker-LAMP][logo]
Docker-LAMP is a set of docker images that include the phusion baseimage (18.04 and 20.04 varieties), along with a LAMP stack ([Apache][apache], [MySQL][mysql] and [PHP][php]) all in one handy package.

With Ubuntu **20.04** and **18.04** images on the `latest-2004` and `latest-1804` tags, Docker-LAMP is flexible enough to use with all of your LAMP projects.

[![Build Status][shield-build-status]][info-build-status]
[![Docker Hub][shield-docker-hub]][info-docker-hub]
[![License][shield-license]][info-license]

### Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [](#)
    - [Contents](#contents)
  - [Introduction](#introduction)
  - [Image Versions](#image-versions)
  - [Using the image](#using-the-image)
    - [On the command line](#on-the-command-line)
    - [With a Dockerfile](#with-a-dockerfile)
    - [MySQL Databases](#mysql-databases)
      - [Creating a database](#creating-a-database)
        - [PHPMyAdmin](#phpmyadmin)
        - [Command Line](#command-line)
        - [Initialization script](#initialization-script)
      - [SQL initialization script](#sql-initialization-script)
  - [Adding your own content](#adding-your-own-content)
    - [Adding your app](#adding-your-app)
    - [Persisting your MySQL](#persisting-your-mysql)
    - [Doing both](#doing-both)
      - [`.bash_profile` alias examples](#bash_profile-alias-examples)
        - [Example usage](#example-usage)
  - [Developing the image](#developing-the-image)
    - [Building and running](#building-and-running)
    - [Setup build arch](#setup-build-arch)
    - [Testing](#testing)
    - [One-line testing command](#one-line-testing-command)
      - [`docker-compose -f docker-compose.test.yml -p ci build;`](#docker-compose--f-docker-composetestyml--p-ci-build)
      - [`docker-compose -f docker-compose.test.yml -p ci up -d;`](#docker-compose--f-docker-composetestyml--p-ci-up--d)
      - [`cd tests && ./test.sh;`](#cd-tests--testsh)
      - [`echo "Exited with status code: $?"`](#echo-exited-with-status-code-)
  - [Inspiration](#inspiration)
  - [Contributing](#contributing)
  - [Building / Releasing](#building--releasing)
  - [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction
As a developer, part of my day to day role is to build LAMP applications. I searched in vain for an image that had everything I wanted, up-to-date packages, a simple interface, good documentation and active support. 

To complicate things even further I needed an image, or actually two, that would run my applications on both 14.04 and 16.04. Having two entirely separate workflows didn't make any sense to me, and Docker-LAMP was born.

Designed to be a single interface that just 'gets out of your way', and works on 18.04 and 20.04 with php 7 and 8. You can move between all images without changing how you work with Docker.

## Image Versions
> **NOTE:** [PHP 5.6 is end of life][end-of-life], so the PHP 5 images `mattrayner/lamp:latest-1404-php5` and `mattrayner/lamp:latest-1604-php5` will not receive any updates. Although these images will stay on Docker Hub, we **strongly** recommend updating you applications to PHP 7 or PHP 8.

> **NOTE**: The 14.04, 16.04 and 18.04 variants of this image are no longer being actively supported or updated.

There are four main 'versions' of the docker image. The table below shows the different tags you can use, along with the PHP, MySQL and Apache versions that come with it.

Component | `latest-1404` | `latest-1604` | `latest-1804-php7` `latest-1804-php8` | `latest-2004-php7` `latest-2004-php8`
---|---|---|---|---
[Apache][apache] | `2.4.7` | `2.4.18` | `2.4.29` | `2.4.41`
[MySQL][mysql] | `5.5.62` | `5.7.30` | `5.7.35` | `8.0.36`
[PHP][php] | `7.3.3` | `7.4.6` | `7.4.23`/`8.0.10` | `7.4.33`/`8.0.30`
[phpMyAdmin][phpmyadmin] | `4.8.5` | `5.0.2` | `5.1.1` | `5.1.1`


## Using the image
### On the command line
This is the quickest way
```bash
# Launch a 20.04 based image with PHP 8
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-2004-php8

# Launch a 20.04 based image with PHP 7
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-2004-php7

# Launch a 18.04 based image with PHP 8
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-1804-php8

# Launch a 18.04 based image with PHP 7
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-1804-php7

# Launch a 16.04 based image with PHP 7
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-1604

# Launch a 14.04 based image with PHP 5
docker run -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest-1404
```

### With a Dockerfile
```docker
FROM mattrayner/lamp:latest-2004-php8

# Your custom commands

CMD ["/run.sh"]
```

### MySQL Databases
By default, the image comes with a `root` MySQL account that has no password. This account is only available locally, i.e. within your application. It is not available from outside your docker image or through phpMyAdmin.

When you first run the image you'll see a message showing your `admin` user's password. This user can be used locally and externally, either by connecting to your MySQL port (default 3306) and using a tool like MySQL Workbench or Sequel Pro, or through phpMyAdmin.

If you need this login later, you can run `docker logs CONTAINER_ID` and you should see it at the top of the log.

#### Creating a database
So your application needs a database - you have three options:

1. PHPMyAdmin
2. Command line
3. Initialization script

##### PHPMyAdmin
Docker-LAMP comes pre-installed with phpMyAdmin available from `http://DOCKER_ADDRESS/phpmyadmin`.

**NOTE:** you cannot use the `root` user with PHPMyAdmin. We recommend logging in with the admin user mentioned in the introduction to this section.

##### Command Line
First, get the ID of your running container with `docker ps`, then run the below command replacing `CONTAINER_ID` and `DATABASE_NAME` with your required values:
```bash
docker exec CONTAINER_ID  mysql -uroot -e "create database DATABASE_NAME"
```

##### Initialization script
See the [SQL initialization script section](#sql-initialization-script) for details.

#### SQL initialization script
Optionally, you can provide a SQL script which will run immediately after MySQL has been installed and configured, allowing you to run custom SQL e.g. to create a database, users or insert custom data.

Please note that **the SQL initialization script runs only at the container first startup**. The script won't run if MySQL has already been configured (i.e. if the `/var/lib/mysql` contains initialized MySQL data).

The below command will run the docker image `mattrayner/lamp:latest` interactively, exposing port `80` on the host machine with port `80` on the docker container. It will also create a volume linking the `script.sql` file within your current folder to the `/db/init.sql` file on the container. This is where the container expects the SQL initialization script to live.

```bash
docker run -i -t -p "80:80" -v ${PWD}/script.sql:/db/init.sql:ro mattrayner/lamp:latest
```

## Adding your own content
The 'easiest' way to add your own content to the lamp image is using Docker volumes. This will effectively 'sync' a particular folder on your machine with that on the docker container.

The below examples assume the following project layout and that you are running the commands from the 'project root'.
```
/ (project root)
/app/ (your PHP files live here)
/mysql/ (docker will create this and store your MySQL data here)
```

In english, your project should contain a folder called `app` containing all of your app's code. That's pretty much it.

### Adding your app
The below command will run the docker image `mattrayner/lamp:latest` interactively, exposing port `80` on the host machine with port `80` on the docker container. It will then create a volume linking the `app/` directory within your project to the `/app` directory on the container. This is where Apache is expecting your PHP to live.
```bash
docker run -i -t -p "80:80" -v ${PWD}/app:/app mattrayner/lamp:latest
```

### Persisting your MySQL
The below command will run the docker image `mattrayner/lamp:latest`, creating a `mysql/` folder within your project. This folder will be linked to `/var/lib/mysql` where all of the MySQL files from container lives. You will now be able to stop/start the container and keep your database changes.

You may also add `-p 3306:3306` after `-p 80:80` to expose the mysql sockets on your host machine. This will allow you to connect an external application such as SequelPro or MySQL Workbench.
```bash
docker run -i -t -p "80:80" -v ${PWD}/mysql:/var/lib/mysql mattrayner/lamp:latest
```

### Doing both
The below command is our 'recommended' solution. It both adds your own PHP and persists database files. We have created a more advanced alias in our `.bash_profile` files to enable the short commands `ldi` and `launchdocker`. See the next section for an example.
```bash
docker run -i -t -p "80:80" -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql mattrayner/lamp:latest
```

#### `.bash_profile` alias examples
The below example can be added to your `~/.bash_profile` file to add the alias commands `ldi` and `launchdocker`. By default it will launch the 16.04 image - if you need the 14.04 image, simply change the `docker run` command to use `mattrayner/lamp:latest-1404` instead of `mattrayner/lamp:latest`.
```bash
# A helper function to launch docker container using mattrayner/lamp with overrideable parameters
#
# $1 - Apache Port (optional)
# $2 - MySQL Port (optional - no value will cause MySQL not to be mapped)
function launchdockerwithparams {
    APACHE_PORT=80
    MYSQL_PORT_COMMAND=""
    
    if ! [[ -z "$1" ]]; then
        APACHE_PORT=$1
    fi
    
    if ! [[ -z "$2" ]]; then
        MYSQL_PORT_COMMAND="-p \"$2:3306\""
    fi

    docker run -i -t -p "$APACHE_PORT:80" $MYSQL_PORT_COMMAND -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql mattrayner/lamp:latest
}
alias launchdocker='launchdockerwithparams $1 $2'
alias ldi='launchdockerwithparams $1 $2'
```

##### Example usage
```bash
# Launch docker and map port 80 for apache
ldi

# Launch docker and map port 8080 for apache
ldi 8080

# Launch docker and map port 3000 for apache along with 3306 for MySQL
ldi 3000 3306
```


## Developing the image
### Building and running
```bash
# Clone the project from Github
git clone https://github.com/mattrayner/docker-lamp.git
cd docker-lamp

# Build the images
docker build --build-arg PHP_VERSION=8.0 -t=mattrayner/lamp:latest -f ./2004/Dockerfile .
docker build --build-arg PHP_VERSION=8.0 -t=mattrayner/lamp:latest-2004-php8 -f ./2004/Dockerfile .
docker build --build-arg PHP_VERSION=7.4 -t=mattrayner/lamp:latest-2004-php7 -f ./2004/Dockerfile .
docker build --build-arg PHP_VERSION=8.0 -t=mattrayner/lamp:latest-1804-php8 -f ./1804/Dockerfile .
docker build --build-arg PHP_VERSION=7.4 -t=mattrayner/lamp:latest-1804-php7 -f ./1804/Dockerfile .

# Run the image as a container
docker run -d -p "3000:80" mattrayner/lamp:latest

# Sleep to allow the container to boot
sleep 30

# Curl out the contents of our new container
curl "http://$(docker-machine ip):3000/"
```
### Setup build arch
On default the image build on arm64 arch using phusion/baseimage:focal-1.1.0, you can use the amd64 if necessary

```bash
docker build --no-cache --build-arg BASE_IMAGE=ubuntu:20.04 -t n3srl/docker-lamp:latest .
```

### Testing
We use `docker-compose` to setup, build and run our testing environment. It allows us to offload a large amount of the testing overhead to Docker, and to ensure that we always test our image in a consistent way that's not affected by the host machine.

### One-line testing command
We've developed a single-line test command you can run on your machine within the `docker-lamp` directory. This will test any changes that may have been made, as well as comparing installed versions of Apache, MySQL, PHP and phpMyAdmin against those expected.
```bash
docker-compose -f docker-compose.test.yml -p ci build; docker-compose -f docker-compose.test.yml -p ci up -d; cd tests && ./test.sh; echo "Exited with status code: $?";
```

So what does this command do?

#### `docker-compose -f docker-compose.test.yml -p ci build;`
First, build that latest version of our docker-compose images.

#### `docker-compose -f docker-compose.test.yml -p ci up -d;`
Launch our docker containers (`web2004-php8` etc.) in daemon mode.

#### `cd tests && ./test.sh;`
Change into the test directory and run out tests

#### `echo "Exited with status code: $?"`
Report back whether the tests passed or not


## Inspiration
This image was originally based on [dgraziotin/lamp][dgraziotin-lamp], with a few changes to make it compatible with the Concrete5 CMS.

I also changed the setup to create ubuntu (well, baseimage, but you get what I'm saying) images so that this project could be as useful as possible to as many people as possible.


## Contributing
If you wish to submit a bug fix or feature, you can create a pull request and it will be merged pending a code review.

1. Clone/fork it
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Test your changes using the steps in [Testing](#testing)
5. Push to the branch (git push origin my-new-feature)
6. Create a new Pull Request

## Building / Releasing
Manually building and releasing can be done with the following:

```bash
docker-compose -f docker-compose.test.yml -p ci build
docker tag ci-web2004-php8 mattrayner/lamp:latest
docker tag ci-web2004-php8 mattrayner/lamp:latest-2004
docker tag ci-web2004-php8 mattrayner/lamp:latest-2004-php8
docker tag ci-web2004-php7 mattrayner/lamp:latest-2004-php7

docker push mattrayner/lamp:latest
docker push mattrayner/lamp:latest-2004
docker push mattrayner/lamp:latest-2004-php8
docker push mattrayner/lamp:latest-2004-php7
```

## License
Docker-LAMP is licensed under the [Apache 2.0 License][info-license].


[logo]: https://cdn.rawgit.com/mattrayner/docker-lamp/831976c022782e592b7e2758464b2a9efe3da042/docs/logo.svg

[apache]: http://www.apache.org/
[mysql]: https://www.mysql.com/
[php]: http://php.net/
[phpmyadmin]: https://www.phpmyadmin.net/

[end-of-life]: http://php.net/supported-versions.php

[info-build-status]: https://circleci.com/gh/mattrayner/docker-lamp
[info-docker-hub]: https://hub.docker.com/r/mattrayner/lamp
[info-license]: LICENSE

[shield-build-status]: https://img.shields.io/circleci/project/mattrayner/docker-lamp.svg
[shield-docker-hub]: https://img.shields.io/badge/docker%20hub-mattrayner%2Flamp-brightgreen.svg
[shield-license]: https://img.shields.io/badge/license-Apache%202.0-blue.svg

[dgraziotin-lamp]: https://github.com/dgraziotin/osx-docker-lamp
