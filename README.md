# MongoDB Stack

_MongoDB Stack_ is a project that provides a collection of examples that show how to run _MongoDB_ in development using _[Docker]_ containers and _[Docker-Compose]_. Each example is represented as a separate stack. And each stack can be run using either the _docker CLI_ or a _docker-compose_ file.

- [Basic](#mongodb-stack)
  
  Host a single node MongoDB instance using _docker_ and _docker-compose_.

- [Using Bind Mounts](#mongodb-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure a bind mount for MongoDB database files.

- [Using Environment Variables](#mongodb-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Use environment variables to specify authentication information and initial database. Specify a javascript file that will be executed as part of setup and initialization (uses entry point scripts)

- [Using Configuration File](#mongodb-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure bind mounts for MongoDB database files and MongoDB configuration file.

- [Using Configuration File With Authorization](#mongodb-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure bind mounts for MongoDB database files and MongoDB configuration file. Specify authorization in MongoDB configuration file.

- [3-Node MongoDB Replica Set](#mongodb-stack)
  
  Host a 3-node MongoDB replica set using _docker_ and _docker-compose_.

[Docker]: https://docs.docker.com/
[Docker-Compose]: https://docs.docker.com/compose/
