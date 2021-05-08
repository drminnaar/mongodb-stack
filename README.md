# MongoDB Stack

_MongoDB Stack_ is a project that provides a collection of examples that show how to run _MongoDB_ in development using _[Docker]_ containers and _[Docker-Compose]_. Each example is represented as a separate stack. And each stack can be run using either the _docker CLI_ or a _docker-compose_ file.

- [Basic](https://github.com/drminnaar/mongodb-stack/tree/master/basic-stack)
  
  Host a single node MongoDB instance using _docker_ and _docker-compose_.

- [Using Bind Mounts](https://github.com/drminnaar/mongodb-stack/tree/master/bindmounts-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure a bind mount for MongoDB database files.

- [Using Environment Variables](https://github.com/drminnaar/mongodb-stack/tree/master/env-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Use environment variables to specify authentication information and initial database. Specify a javascript file that will be executed as part of setup and initialization (uses entry point scripts)

- [Using Configuration File](https://github.com/drminnaar/mongodb-stack/tree/master/conf-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure bind mounts for MongoDB database files and MongoDB configuration file.

- [Using Configuration File With Authorization](https://github.com/drminnaar/mongodb-stack/tree/master/conf-with-auth-stack)

  Host a single node MongoDB instance using _docker_ and _docker-compose_. Configure bind mounts for MongoDB database files and MongoDB configuration file. Specify authorization in MongoDB configuration file.

- [3-Node MongoDB Replica Set](https://github.com/drminnaar/mongodb-stack/tree/master/basic-repl-stack)
  
  Host a 3-node MongoDB replica set using _docker_ and _docker-compose_.

- [Database Seed](https://github.com/drminnaar/mongodb-stack/tree/main/seed-stack)

  Host a single MongoDB instance and seed the database with some initial data. We use the sample _MFlix_ database that is available from [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) to seed the database.

---

[Docker]: https://docs.docker.com/
[Docker-Compose]: https://docs.docker.com/compose/
[MongoDB Docs]: https://docs.mongodb.com
[MongoDB Atlas]: https://www.mongodb.com/cloud/atlas
[MongoDB Documentation]: https://docs.atlas.mongodb.com
[Daas]: https://en.wikipedia.org/wiki/Data_as_a_service
[MongoDB Compass]: https://www.mongodb.com/products/compass
