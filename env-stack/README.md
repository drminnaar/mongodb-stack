# MongoDB Setup Using Environment Variables

Host a single node MongoDB instance using _docker_ and _docker-compose_. Use environment variables to specify authentication information and initial database. Specify a javascript file that will be executed as part of setup and initialization (uses entry point scripts)

---

## Contents

- [Description](#description)
- [Command Line Setup](#command-line-setup)
- [Docker-Compose Setup](#docker-compose-setup)
- [Connect](#connect)

---

## Description

This document describes 2 different ways in which to setup and host a single node MongoDB instance. The first way is to use the _docker CLI_ by executing a number of _docker_ commands. The second way is to use a _docker-compose_ file that describes the MongoDB stack. For both methods, 2 bind mounts are created for the MongoDB database files and the MongoDB configuration file.

Summary:

- Host single node MongoDB instance using _docker CLI_
- Host single node MongoDB instance using _docker-compose_
- Specify volume mount using `--volume mongo-data-db:/data/db`
- Specify bind mount using `--volume ~/mongo-stack/scripts/events.js:/docker-entrypoint-initdb.d/events.js:ro`
- Specify environment variables:
  - MONGO_INITDB_ROOT_USERNAME
  - MONGO_INITDB_ROOT_PASSWORD
  - MONGO_INITDB_DATABASE
- Connect to MongoDB instance
- Cleanup of stack

Prerequisites:

- Must have MongoDB client tools installed

  ```bash
  # Verify version of installed MongoDB client (mongo). Version 4 is recommended
  $ mongo --version

  MongoDB shell version v4.4.1
  Build Info: {
      "version": "4.4.1",
      "gitVersion": "ad91a93a5a31e175f5cbf8c69561e788bbc55ce1",
      "openSSLVersion": "OpenSSL 1.1.1f  31 Mar 2020",
      "modules": [],
      "allocator": "tcmalloc",
      "environment": {
          "distmod": "ubuntu2004",
          "distarch": "x86_64",
          "target_arch": "x86_64"
      }
  }
  ```

- Must have local directory created for initialization scripts. The `events.js` script that accompanies this stack must be copied to the aforementioned scripts directory

  ```bash
  # Create scripts folder
  mkdir -p ~/mongo-stack/scripts

  # Copy events.js to scripts folder
  cp ./env-stack/events.js ~/mongo-stack/scripts
  ```

---

## Command Line Setup

### Run Single Node MongoDB Instance

```bash
# Create docker volume
docker volume create mongo-data-db

# Create docker network that will be used by MongoDB container
docker network create mongo-net

# Run MongoDB container specifying environment variables
# Take note of the 'docker-entrypoint-initdb.d' bind mount
docker container run \
    --detach \
    --name mongo-1 \
    --publish 27017:27017 \
    --restart unless-stopped \
    --network mongo-net \
    --volume mongo-data-db:/data/db \
    --volume ~/mongo-stack/scripts/events.js:/docker-entrypoint-initdb.d/events.js:ro \
    --env MONGO_INITDB_ROOT_USERNAME=root \
    --env MONGO_INITDB_ROOT_PASSWORD=password \
    --env MONGO_INITDB_DATABASE=events_db \
    mongo:4.4-bionic
```

### Cleanup

```bash
# Remove MongoDB container
docker container rm mongo-1 --force

# Remove docker network
docker network rm mongo-net

# Remove docker volume
docker volume rm mongo-data-db
```

---

## Docker-Compose Setup

### Create 'docker-compose' Stack

```yaml
version: "3.7"
services:
  mongo-1:
    image: mongo:4.4-bionic
    container_name: mongo-1
    restart: unless-stopped
    ports:
      - 27017:27017
    networks:
      - default
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=password
      - MONGO_INITDB_DATABASE=events_db
    volumes:
      - type: volume
        source: mongo-data-db
        target: /data/db
      - type: bind
        source: ~/mongo-stack/scripts/events.js
        target: /docker-entrypoint-initdb.d/events.js
        read_only: true
networks:
  default:
    name: mongo-net
    driver: bridge
volumes:
  mongo-data-db:
```

### Run 'docker-compose' Stack

```bash
# Run MongoDB stack as a daemon
docker-compose up -d
```

### Cleanup

```bash
# Remove MongoDB stack including the removal of volumes
docker-compose down -v
```

---

## Connect

```bash
# Connect to MongoDB instance
mongo --host localhost --port 27017 --username root --password password --authenticationDatabase admin

# Connect to events_db database and display events
show dbs
use events_db
show collections
db.events.find().pretty()
```
