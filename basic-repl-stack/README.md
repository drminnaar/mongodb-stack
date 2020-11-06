# MongoDB Replication Setup

Host a 3-node MongoDB replica set using _docker_ and _docker-compose_.

---

## Contents

- [Description](#description)
- [Command Line Setup](#command-line-setup)
- [Docker-Compose Setup](#docker-compose-setup)
- [Configure Replica Set](#configure-replica-set)
- [Connect To Replica Set](#connect-to-replica-set)

---

## Description

This document describes 2 different ways in which to setup and host a 3-node MongoDB replica set. The first way is to use the _docker CLI_ by executing a number of _docker_ commands. The second way is to use a _docker-compose_ file that describes the MongoDB replica set. Before using any of the described methods, it is important to add a number of host entries to ones local machine hosts file.

Summary:

- Host 3-node MongoDB replica set using _docker CLI_
- Host 3-node MongoDB replica set using _docker-compose_
- Connect to stack using `mongo --host <replica-set-name>/<node1>,<node2>,<node3>` format
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

- Must update local machine host file. Append the following host file entry to the end of local hosts file

  ```bash  
  # MongoDB replica set node host entries
  127.0.0.1        mongo-1, mongo-2, mongo-3
  ```

  Note:

  - For Linux, update `/etc/hosts` file
  - For Windows, update `c:\windows\system32\drivers\etc\hosts` file

---

## Command Line Setup

### Run 3 MongoDB Nodes

```bash
# Create docker network that will be used by MongoDB containers
docker network create mongo-net

# Run Primary MongoDB container
docker container run \
  --detach \
  --name mongo-1 \
  --publish 27011:27011 \
  --restart unless-stopped \
  --net mongo-net \
  mongo:4.4-bionic \
  mongod --port 27011 --replSet mongo-repl-set

# Run Secondary MongoDB container
docker container run \
    --detach \
    --name mongo-2 \
    --publish 27012:27012 \
    --restart unless-stopped \
    --net mongo-net \
    mongo:4.4-bionic \
    mongod --port 27012 --replSet mongo-repl-set

# Run Secondary MongoDB container
docker container run \
    --detach \
    --name mongo-1 \
    --publish 27013:27013 \
    --restart unless-stopped \
    --net mongo-net \
    mongo:4.4-bionic \
    mongod --port 27013 --replSet mongo-repl-set
```

### Cleanup

```bash
# Remove MongoDB containers
docker container rm $(docker container ls --filter name=mongo-* -q) --force

# Remove docker network
docker network rm mongo-net
```

---

## Docker-Compose Setup

### Create 'docker-compose.yml' file

```yaml
version: "3.7"
services:
  # node 1 in replica set
  mongo-1:
    image: mongo:4.4-bionic
    container_name: mongo-1
    restart: unless-stopped
    ports:
      - 27011:27011
    networks:
      - default
    command: ["--port", "27011", "--replSet", "mongo-repl-set"]
  # node 2 in replica set
  mongo-2:
    image: mongo:4.4-bionic
    container_name: mongo-2
    restart: unless-stopped
    ports:
      - 27012:27012
    networks:
      - default
    command: ["--port", "27012", "--replSet", "mongo-repl-set"]
  # node 3 in replica set
  mongo-3:
    image: mongo:4.4-bionic
    container_name: mongo-3
    restart: unless-stopped
    ports:
      - 27013:27013
    networks:
      - default
    command: ["--port", "27013", "--replSet", "mongo-repl-set"]
networks:
  default:
    name: mongo-net
    driver: bridge
```

### Run 'docker-compose' Stack

```bash
# Run MongoDB stack as daemon
docker-compose up -d
```

### Cleanup

```bash
# Remove MongoDB stack including the removal of volumes
docker-compose down -v
```

## Configure Replica Set

```bash
# Connect to 'mongo-1'
docker container exec -it mongo-1 mongo --port 27011

# Define replica set configuration
let rsConfig = {
  "_id": "mongo-repl-set",
  "members": [
    {
      "_id": 0,
      "host": "mongo-1:27011"
    },
    {
      "_id": 1,
      "host": "mongo-2:27012"
    },
    {
      "_id": 2,
      "host": "mongo-3:27013"
    }
  ]
}

# Initiate replica set using configuration
rs.initiate(rsConfig)

# Return the replica set status from this node's context
rs.status()

# Return a document that describes the role of this mongod instance
rs.isMaster()

# Return a document that contains the current replica set configuration
rs.config()
```

## Connect To Replica Set

```bash
# Connect to MongoDB replica set
mongo --host mongo-repl-set/mongo-1:27011,mongo-2:27012,mongo-3:27013 --quiet
```
