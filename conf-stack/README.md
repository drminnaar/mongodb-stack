# MongoDB Setup Using Configuration

Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure bind mounts for MongoDB database files and MongoDB configuration file.

---

## Contents

- [Description](#description)
- [Create Configuration File](#create-configuration-file)
- [Command Line Setup](#command-line-setup)
- [Docker-Compose Setup](#docker-compose-setup)
- [Connect](#connect)

---

## Description

This document describes 2 different ways in which to setup and host a single node MongoDB instance. The first way is to use the _docker CLI_ by executing a number of _docker_ commands. The second way is to use a _docker-compose_ file that describes the MongoDB stack. For both methods, 2 bind mounts are created for the MongoDB database files and the MongoDB configuration file.

Summary:

- Host single node MongoDB instance using _docker CLI_
- Host single node MongoDB instance using _docker-compose_
- Specify bind mount using `--volume ~/mongo-stack/data/db:/opt`
- Specify bind mount using `--volume ~/mongo-stack/config/mongod.conf:/etc/mongod.conf:ro`
- Create a minimal MongoDB configuration file (`mongod.conf`)
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

- Must have local directories created for both MongoDB database files and MongoDB configuration

  ```bash
  # Create a directory to hold MongoDB data
  mkdir -p ~/mongo-stack/data/db

  # Create directory to hold MongoDB configuration file
  mkdir -p ~/mongo-stack/config
  ```

---

## Create Configuration File

```bash
vim ~/mongo-stack/config/mongod.conf
```

```yml
# Add the following configuration
# Note that we specify the '/opt' directory for dbPath instead
# of default '/data/db'. This is for illustration purposes.
storage:
  dbPath: "/opt"
  journal:
    enabled: true
```

---

## Command Line Setup

### Run Single Node MongoDB Instance

```bash
# Create docker network that will be used by MongoDB container
docker network create mongo-net

# Run MongoDB container specifying configuration file
docker container run \
    --detach \
    --name mongo-1 \
    --publish 27017:27017 \
    --restart unless-stopped \
    --network mongo-net \
    --volume ~/mongo-stack/data/db:/opt \
    --volume ~/mongo-stack/config/mongod.conf:/etc/mongod.conf:ro \
    mongo:4.4-bionic mongod --config /etc/mongod.conf
```

### Cleanup

```bash
# Remove MongoDB container
docker container rm mongo-1 --force

# Remove docker network
docker network rm mongo-net

# Remove database files (may need sudo)
rm -r ~/mongo-stack/data/db/*

# Remove config file (may need sudo)
rm ~/mongo-stack/config/mongod.conf
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
    volumes:
      - type: bind
        source: ~/mongo-stack/data/db
        target: /opt
      - type: bind
        source: ~/mongo-stack/config/mongod.conf
        target: /etc/mongod.conf
        read_only: true
    command: mongod --config /etc/mongod.conf
networks:
  default:
    name: mongo-net
    driver: bridge
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

# Remove database files (may need sudo)
rm -r ~/mongo-stack/data/db/*

# Remove config file (may need sudo)
rm ~/mongo-stack/config/mongod.conf
```

---

## Connect

```bash
# Connect to MongoDB instance
mongo --host localhost --port 27017
```
