# MongoDB Setup With Seed Data

Host a single MongoDB instance and seed the database with some initial data. We use the sample _MFlix_ database that is available from [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) to seed the database.

## MongoDB Seed

The following 2 files are used as part of seeding the _MFlix_ database:

- mflix.gz
- Dockerfile

### MFlix Database File (mflix.gz)

The file `mflix.gz` is a compressed mongodb dump of the _MFlix_ database. The _MFlix_ database is a sample database that is provided as part of the [MongoDB Atlas]. [MongoDB Atlas] is a _Data as a Service (DaaS)_ offering, and is hosted in the cloud. There is no installation of MongoDB required and a free tier is available.

I decided to make it even easier to work with this project by obtaining the free _MFlix_ sample dataset and including it as part of the Docker stack. However, if you would like to obtain the original _MFlix_ database, you can do so by following the following steps:

- To get started, signup for free by registering for a free tier account [here](https://www.mongodb.com/cloud/atlas). The free tier entitles you to 512MB storage. Please review the [MongoDB Atlas Documentation] for more information.
- Once you have registered and setup your MongoDB instance on _Atlas_, you will be presented with a dashboard resembling the following image:

  ![atlas](https://user-images.githubusercontent.com/33935506/37251882-d19a3818-2520-11e8-97f6-7015435f3cbd.png)

- Select the ellipses next to the `Connect` button. Then select `Load Sample Dataset`

  ![mflix-atlast-setup-2](https://user-images.githubusercontent.com/33935506/111863748-40473000-89c2-11eb-8c5a-8706195d7048.png)

- Take note of sample dataset size. Select `Load Sample Dataset`.

  ![mflix-atlast-setup-3](https://user-images.githubusercontent.com/33935506/111863746-3f160300-89c2-11eb-8409-dc0465dfc3ed.png)

- Open your terminal (bash/powershell) and create a dump of the _MFlix_ database. Take note that in addition to creating the dump, I also compress the file.

  ```bash
  # dump the mflix database to a local file
  mongodump --uri="mongodb+srv://your_mongodb_atlas_cluster" --db=sample_mflix --username=mongo --gzip --archive=mflix.gz
  ```

- In the same terminal window, run the following command to restore the dump to a destination of your choice. Take note that in the command below I am renaming the `sample_mflix` database to just `mflix` database as part of restore.

  ```bash
  # restore mflix database from file
  mongorestore --host=localhost --port=27017 --username=dbadmin --nsFrom="sample_mflix.*" --nsTo="mflix.*" --gzip --archive=mflix.gz
  ```

### MongoDB Seed Dockerfile

This Dockerfile will be used as part of the `docker-compose` stack and is used to seed the MongoDB database. It runs the commands to be able to copy the `mflix.gz` file and restore it to the Docker hosted MongoDB database.

```docker
FROM mongo:4.4-bionic
COPY mflix.gz mflix.gz
CMD mongorestore --host=mflix-mongo --port=27017 --username=dbadmin --password=password --nsFrom="sample_mflix.*" --nsTo="mflix.*" --gzip --archive=mflix.gz
```

---

## The docker-compose File

The `docker-compose.yaml` file defines the mongodb stack for this example. The stack is defined as follows:

- Define MongoDB Setup
  
  - The service is defined as follows:

    ```yaml
    # MongoDB Setup
    mflix-mongo:
      image: mongo:4.4-bionic
      container_name: mflix-mongo
      restart: unless-stopped
      ports:
        - 27017:27017
      networks:
        - default
      environment:
        - MONGO_INITDB_ROOT_USERNAME=dbadmin
        - MONGO_INITDB_ROOT_PASSWORD=password
      volumes:
        - type: volume
          source: mflix-mongo-data
          target: /data/db
    ```

- Define MongoDB Seed Setup
  
  - Depends on MongoDB Setup to complete first
  - The service is defined as follows:

    ```yaml
    # Container to seed MongoDB with MFlix data
    mflix-mongo-seed:
      container_name: mflix-mongo-seed
      build: ./mongo-seed
      networks:
        - default
      depends_on:
        - mflix-mongo
    ```

## Manage Stack

### Start Stack

```bash
# Remove MongoDB stack including the removal of volumes
docker-compose up --detach
```

### Cleanup Stack

```bash
# Remove MongoDB stack including the removal of volumes
docker-compose down -v
```

---

[MongoDB Docs]: https://docs.mongodb.com
[Docker]: https://www.docker.com
[MongoDB Atlas]: https://www.mongodb.com/cloud/atlas
[MongoDB Documentation]: https://docs.atlas.mongodb.com
[Daas]: https://en.wikipedia.org/wiki/Data_as_a_service
[MongoDB Compass]: https://www.mongodb.com/products/compass