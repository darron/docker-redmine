# Docker Redmine

Dockerfile to build a Redmine container image (with some additional themes and plugins).

## Installation

```bash
git clone https://github.com/sameersbn/docker-redmine.git
cd docker-redmine
sudo docker build -t="redmine" .
```

## Quick Start
Run the redmine image

```bash
REDMINE=$(sudo docker run -d redmine)
REDMINE_IP=$(sudo docker inspect $REDMINE | grep IPAddres | awk -F'"' '{print $4}')
```

Access the Redmine application

```bash
xdg-open "http://${REDMINE_IP}"
```

__NOTE__: Please allow a minute or two for the Redmine application to start.

Login using the default username and password:

* username: admin
* password: admin

You should now have Redmine ready for testing. If you want to use Redmine for more than just testing then please read the **Advanced Options** section.

## Advanced Options

### Mounting volumes
For the file storage we need to mount a volume at the following location.

* /redmine/files

Volumes can be mounted in docker by specifying the **'-v'** option in the docker run command.

```bash
mkdir -pv /opt/redmine/files
docker run -d -v /opt/redmine/files:/redmine/files redmine
```

### Configuring MySQL database connection
Redmine uses a database backend to store its data.

#### Using the internal mysql server
This docker image is configured to use a MySQL database backend. The database connection can be configured using environment variables. If not specified, the image will start a mysql server internally and use it. However in this case, the data stored in the mysql database will be lost if the container is stopped/deleted. To avoid this you should mount a volume at /var/lib/mysql.

```bash
mkdir /opt/redmine/mysql
docker run -d \
  -v /opt/redmine/files:/redmine/files \
  -v /opt/redmine/mysql:/var/lib/mysql redmine
```

This will make sure that the data stored in the database is not lost when the image is stopped and started again.

#### Using an external mysql server
The image can be configured to use an external MySQL database instead of starting a MySQL server internally. The database configuration should be specified using environment variables while starting the Redmine image.

Before you start the Redmine image create user and database for redmine.

```bash
mysql -uroot -p
CREATE USER 'redmine'@'%.%.%.%' IDENTIFIED BY 'password';
CREATE DATABASE IF NOT EXISTS `redmine_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `redmine_production`.* TO 'redmine'@'%.%.%.%';
```

To make sure the database is initialized start the container with DB_INIT=yes environment variable set.

**NOTE: This should be done only for the first run**.

*Assuming that the mysql server host is 192.168.1.100*

```bash
docker run -d \
  -e "DB_HOST=192.168.1.100" -e "DB_NAME=redmine_production" \
  -e "DB_USER=redmine" -e "DB_PASS=password" -e "DB_INIT=yes" \
  -v /opt/redmine/files:/redmine/files redmine
```

This will initialize the redmine database. Now that the database is initialized, omit the **-e "DB_INIT=yes"** option from the docker command.

```bash
docker run -d \
  -e "DB_HOST=192.168.1.100" -e "DB_NAME=redmine_production" \
  -e "DB_USER=redmine" -e "DB_PASS=password" \
  -v /opt/redmine/files:/redmine/files redminee
```

### Other options
Below is the complete list of parameters that can be set using environment variables.

* DB_HOST

        The mysql server hostname. Defaults to localhost.

* DB_NAME

        The mysql database name. Defaults to redmine_production

* DB_USER

        The mysql database user. Defaults to root

* DB_PASS

        The mysql database password. Defaults to no password

* DB_POOL

        The mysql database connection pool count. Defaults to 5.

* DB_INIT

        Whether to initial the mysql database. Defaults to no

* PASSENGER_MAX_POOL_SIZE

        PassengerMaxPoolSize (default: 6)

* PASSENGER_MIN_INSTANCES

        PassengerMinInstances (default: 1)

* PASSENGER_MAX_REQUESTS

        PassengerMaxRequests (default: 0)

* PASSENGER_POOL_IDLE_TIME

        PassengerPoolIdleTime (default: 300)

### Putting it all together

```bash
docker run -d -h redmine.local.host \
  -v /opt/redmine/files:/redmine/files \
  -v /opt/redmine/mysql:/var/lib/mysql \
  redmine
```

If you are using an external mysql database

```bash
docker run -d -h redmine.local.host \
  -v /opt/redmine/files:/redmine/files \
  -e "DB_HOST=192.168.1.100" -e "DB_NAME=redmine_production" -e "DB_USER=redmine" -e "DB_PASS=password" \
  redmine
```

## References
  http://www.redmine.org/

  http://www.redmine.org/projects/redmine/wiki/Guide

  http://www.redmine.org/projects/redmine/wiki/RedmineInstall
