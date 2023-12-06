---
title: "How to install Apache Guacamole in Docker"
date: 2023-12-05
draft: false
description: "How to install Apache Guacamole in Docker"
tags:
  - linux
  - docker
  - apache guacamole
cover: /img/guacamole-logo.jpg
---

[Apache Guacamole][guac] is a web-based RDP, VNC, and SSH client. You can use it
to access your servers from a web browser, without requiring any software
to be installed on client devices. The installation process of Guacamole is
relatively straightforward and takes a matter of minutes when using Docker.

Before proceeding with the installation of Guacamole, the Docker
Engine must be installed on your server. A guide for installing Docker on Debian
is available [here][docker-install].

## Create the Docker configuration

1.  Create a new directory for the Guacamole Docker compose file, and then enter
    the directory.

```bash
mkdir ~/guac
cd ~/guac
```

2.  Create a new environment variable file, `.env`. This file will contain
    environment variables that store the database connection information.

```
GUACAMOLE_DATABASE=guacamole
GUACAMOLE_USER=guac
GUACAMOLE_PASSWORD=your-password-here
```

3.  Create a new `docker-compose.yml` file with the following contents:

```yaml
---
version: '3'

services:
  guacd:
    restart: always
    image: guacamole/guacd

  guacamole-server:
    restart: always
    image: guacamole/guacamole
    links:
      - guacd
      - database
    ports:
      - "8080:8080"
    environment:
      - GUACD_HOSTNAME=guacd
      - POSTGRESQL_HOSTNAME=database
      - POSTGRESQL_DATABASE=${GUACAMOLE_DATABASE}
      - POSTGRESQL_USER=${GUACAMOLE_USER}
      - POSTGRESQL_PASSWORD=${GUACAMOLE_PASSWORD}

  database:
    restart: always
    image: postgres:15.5-alpine
    environment:
      - POSTGRES_USER=${GUACAMOLE_USER}
      - POSTGRES_DB=${GUACAMOLE_DATABASE}
      - POSTGRES_PASSWORD=${GUACAMOLE_PASSWORD}
    volumes:
      - ./db:/var/lib/postgresql/data:rw
      - ./initdb.sql:/initdb.sql:ro
```

## Configure the PostgreSQL database

1. Generate the database schema in PostgreSQL syntax.

```bash
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > initdb.sql
```

2.  Start all the docker containers.

```bash
docker compose up -d
```

3.  Enter the PostgreSQL container and configure the database.

```bash
# Get a shell as the postgres user on the PostgreSQL container
docker exec -u postgres -it guac-database-1 /bin/sh

# Import the database schema
psql -U guac guacamole -f /initdb.sql

# Exit the shell
exit
```

4.  Open the `docker-compose.yml` file once more and remove the line for
    mounting the `initdb.sql` file to the PostgreSQL container.

5.  Recreate the containers

```bash
docker compose up -d
```

## Configuring Guacamole

Open a web browser, and navigate to http://your.ip:8080 and you should be
presented with a login screen. The default username and password for Guacamole
is `guacadmin`. 

Once you have signed in to Guacamole, it is recommended to change the
administrator password. This can be done by following the below steps:

1. Click your username in the top right corner of the screen
2. Click **Settings**, and then go to the **Preferences** tab.
3. Type in your current password, and then your new password.
4. Click **Update Password** to update your password.

Once your password has been changed, you can add additional users and begin
[adding connections][connections]!


{{< figure
  src="/img/guacamole-screenshot.png"
  alt="Apache Guacamole screenshot"
  caption="Apache Guacamole in use"
  position="center" >}}

[guac]:http://guacamole.apache.org/
[docker-install]:https://docs.docker.com/engine/install/debian/
[connections]:https://guacamole.apache.org/doc/gug/configuring-guacamole.html#configuring-connections

