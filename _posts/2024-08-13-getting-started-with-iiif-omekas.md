---
layout: post
read_time: true
show_date: true
title: Getting started with IIIF and Omeka S
description: How to get Omeka S and IIIF running with Docker
date: 2024-08-13
img: posts/docker-omekas-iiif.jpg
tags: [ omeka s, iiif, docker ]
author: Maarten Coonen
---

In our [previous post](./um-library-participates-in-nde-versnellen.html) we described the "NDE Versnellen 2023"
project in which we planned to make our Omeka S instance fully NDE compatible by implementing the IIIF-functionality on
top of it.

This post presents the results of that project by guiding you through an easy-to-setup Dockerized environment consisting
of Omeka S with IIIF preconfigured. We made this Dockerized Omeka S environment freely available under a GPL-3.0 open
source license.

**Please note that this Dockerized environment was designed for development purposes. We would not recommend production
usage in its current state! Read our [next post](./installing-iiif-with-omekas.html) with deployment instructions instead.**

### Before you start

In order to run our Dockerized Omeka S, you need a system with the following characteristics and software installed.

- A supported operating system, such as various Linux distros or MacOS. Details
  on [Docker website](https://docs.docker.com/engine/install/)
- Docker & Docker Compose
- Git

### Create a working directory

```bash
cd ~
mkdir work
cd ~/work
```

### Clone our git repositories

Load our software from the Maastricht University Library GitHub. The steps below are based on
this [readme](https://github.com/MaastrichtU-Library/omekas-docker/blob/master/README.md) and repeated here for
convenience.

In order to resolve DNS requests to the local containers, you need to add the following line to your local `/etc/hosts` file.
```bash
127.0.0.1	localhost omeka.local solr.local db.local iipsrv.local cantaloupe.local viewer.local
```

Download the Omeka S Docker project:
```bash
git clone https://github.com/MaastrichtU-Library/omekas-docker.git
```

Download the Omeka S theme and helper scripts
```bash
cd ~/work/omekas-docker/externals
git clone https://github.com/MaastrichtU-Library/omekas-helpers.git
git clone https://github.com/MaastrichtU-Library/omekas-theme-um.git
```


Set database connection settings

1. Edit the `~/work/omekas-docker/docker-compose.yml` file and enter values for:
    ```bash
    MYSQL_ROOT_PASSWORD:
    MYSQL_DATABASE: 
    MYSQL_USER:
    MYSQL_PASSWORD:
    ```

1. Copy the example database config for Omeka S
    ```bash
    cp ~/work/omekas-docker/omeka-s/config/example_database.ini ~/work/omekas-docker/omeka-s/config/database.ini
    ```

1. Edit the `database.ini` file with a text editor and enter the same values for:
    ```bash
    user     = 
    password = 
    dbname   = 
    host     = 
    ```

### Run Omeka S in Docker
Build the Omeka S infra with:
```bash
docker compose build
```

Additionally, to build the external IIIF-tooling, use:
```bash
docker compose -f docker-compose-iiif-external.yml build
```

Run the Omeka S infra with:
```bash
docker compose up -d
```

To run external IIIF-tooling as well, use:
```bash
docker compose -f docker-compose-iiif-external.yml up -d
```

To see logs, use:
```bash
docker compose logs -f
```

There are some manual configuration steps to perform after the Docker containers have started.
- [Configure Solr backend](https://github.com/MaastrichtU-Library/omekas-docker/blob/master/README-02-Solr.md)

When all containers have started, you will find the services at the following URLs:
- **Omeka S:** http://omeka.local _(usr/pwd: admin@example.org / foobar)_
- **mySQL backend:** http://db.local
- **Solr backend:** http://solr.local
- **IIP IIIF server:** http://iipsrv.local
- **Cantaloupe IIIF server:** http://cantaloupe.local _(usr/pwd: admin / foobar)_
- **Mirador external IIIF viewer:** http://viewer.local



### Well done
If everything worked out, you now have a working Omeka S with IIIF features running locally. 
Stay tuned for our [next post](./installing-iiif-with-omekas.html) with instructions for production usage.