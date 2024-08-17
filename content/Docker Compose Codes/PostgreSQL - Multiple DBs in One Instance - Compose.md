---
title: PostgreSQL - Multiple DBs in One Instance - Compose
date: 2023-11-02 14:54
aliases:
  - example
draft: false
tags:
  - docker
  - compose codes
  - postgre
  - sql
  - multiple
  - databases
  - db
  - single
  - instance
  - script
  - bash
---
 Having multiple DBs served from one instance has pros and cons. The pros is that it's all centralized and is easier to run. The problems are that the performance when accessing can be very up-n-down. Especially if your hosting databases for, lets say Davinci Resolve. I'd recommend giving programs like Resolve its own Postgres instance.

```yaml
version: "2.1"
services:
# PostgreSQL 14:
  postgres14:
    image: postgres:14
    network_mode: bridge
    ports:
      - 5432:5432
    container_name: postgres14
    environment:
      POSTGRES_MULTIPLE_DATABASES: your,new,database,names,here
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - /path/to/appdata/postgresql_scripts/:/docker-entrypoint-initdb.d
      - /path/to/appdata/postgresql:/var/lib/postgresql/data
```

`POSTGRES_MULTIPLE_DATABASES` This variable holds all the names of your databases. Note, this is ONLY used if your database folder directory is empty. It will NOT overwrite existing DBs.

`Postgresql_scripts` This directory will contain the "__init-user-db.sh__" script we will create below. ​

---

### The Script

In the directory `postgresql_scripts` your gonna create a shell script named "__init-user-db.sh__". Paste this code inside it:

```bash
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
	CREATE USER docker;
	CREATE DATABASE docker;
	GRANT ALL PRIVILEGES ON DATABASE docker TO docker;
EOSQL
```

This script will check the "__POSTGRES_MULTIPLE_DATABASES"__ variable and create databases using the names supplied. If the dbs already exists, it will do nothing.

---

### Refrences

- PostgreSQL at Docker.io  
	[https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres)