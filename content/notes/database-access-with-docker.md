---
title: Database Access with Docker
date: 2025-04-01
summary: Quickly connect to PostgreSQL, MariaDB, MongoDB and Redis databases using Docker with simple, one-line commands, bypassing the need for local installations.
categories: [Docker, PostgreSQL, MariaDB, MongoDB, Redis]
---

{{< admonition type="tip" title="TL;DR" >}}
Quickly connect to [PostgreSQL](https://www.postgresql.org/), [MariaDB](https://mariadb.org/), [MongoDB](https://www.mongodb.com/) and [Redis](https://redis.io/) databases using [Docker](https://www.docker.com/) with simple, one-line commands, bypassing the need for local installations.
{{< /admonition >}}

Ever needed to quickly connect to a database without installing the client tools directly on your machine? **Docker** to the rescue! This allows you to jump into a database shell from any environment with Docker installed, saving time and keeping your system clean.

## PostgreSQL

```shell
docker run --rm -it -e PGPASSWORD=$DB_PASS postgres:17 psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME
```

## MariaDB

```shell
docker run --rm -it mariadb:11 mariadb -h $DB_HOST -P $DB_PORT -u $DB_USER -p"$DB_PASS" -D $DB_NAME
```

## MongoDB

```shell
docker run --rm -it mongo:8 mongosh --host $DB_HOST --port $DB_PORT -u $DB_USER -p $DB_PASS
```

## Redis

```shell
docker run --rm -it redis:7.4 redis-cli -h $DB_HOST -p $DB_PORT
```

{{< admonition type="warning" >}}
Double-check that the Docker **image version** you select aligns with the compatibility requirements of your targeted database server to prevent any version-related issues.
{{< /admonition >}}
