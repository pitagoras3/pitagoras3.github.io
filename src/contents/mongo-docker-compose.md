---
author: Szymon Marcinkiewicz
datetime: 2022-05-29T18:20:00Z
title: "#2: MongoDB on docker-compose - standalone || sharded || replicaSet"
slug: mongodb-on-docker-compose
featured: true
draft: false
tags:
  - mongodb
  - docker
ogImage: ""
description: How to setup standalone, sharded or replicaSet MongoDB using docker-compose.
---

## Standalone

To deploy standalone MongoDB using docker-compose you can follow instructions provided in [MongoDB image on DockerHub](https://hub.docker.com/_/mongo). 

**Result:**

```yaml
version: "3.8"

services:
  mongodb5-standalone:
    image: mongo:5.0
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
```

After running `docker-compose up` on the yaml above, you should be able to connect to standalone MongoDB with uri:

```
mongodb://root:example@localhost:27017
```

## Sharded

To deploy sharded MongoDB using docker-compose you can follow instructions provided in [Bitnami image for sharded MongoDB on DockerHub](https://hub.docker.com/r/bitnami/mongodb-sharded/).

**Result:**

```yaml
version: "3.8"

services:
  mongodb5-sharded:
    hostname: mongodb5-sharded
    image: docker.io/bitnami/mongodb-sharded:5.0
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb5-sharded
      - MONGODB_SHARDING_MODE=mongos
      - MONGODB_CFG_PRIMARY_HOST=mongodb-cfg
      - MONGODB_CFG_REPLICA_SET_NAME=cfgreplicaset
      - MONGODB_REPLICA_SET_KEY=replicasetkey123
      - MONGODB_ROOT_PASSWORD=password123
    ports:
      - "27017:27017"

  mongodb-shard0:
    hostname: mongodb-shard0
    image: docker.io/bitnami/mongodb-sharded:5.0
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb-shard0
      - MONGODB_SHARDING_MODE=shardsvr
      - MONGODB_MONGOS_HOST=mongodb5-sharded
      - MONGODB_ROOT_PASSWORD=password123
      - MONGODB_REPLICA_SET_MODE=primary
      - MONGODB_REPLICA_SET_KEY=replicasetkey123
      - MONGODB_REPLICA_SET_NAME=shard0
    volumes:
      - 'shard0_data:/bitnami'

  mongodb-cfg:
    hostname: mongodb-cfg
    image: docker.io/bitnami/mongodb-sharded:5.0
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb-cfg
      - MONGODB_SHARDING_MODE=configsvr
      - MONGODB_ROOT_PASSWORD=password123
      - MONGODB_REPLICA_SET_MODE=primary
      - MONGODB_REPLICA_SET_KEY=replicasetkey123
      - MONGODB_REPLICA_SET_NAME=cfgreplicaset
    volumes:
      - 'cfg_data:/bitnami'

volumes:
  shard0_data:
    driver: local
  cfg_data:
    driver: local
```

After running `docker-compose up` on the yaml above, you should be able to connect to sharded MongoDB with [mongos](https://www.mongodb.com/docs/manual/sharding/#connecting-to-a-sharded-cluster) uri:

```
mongodb://root:password123@localhost:27017
```

## Replica set

To deploy replica set MongoDB using docker-compose you can follow instructions provided in [Soham Kamani's article](https://www.sohamkamani.com/docker/mongo-replica-set/). Unfortunately I had problems with starting original solution (because of no healtheckeck and missing `/etc/hosts` configuration).

**Result after fixes:**

```yaml
version: "3.8"

services:
  mongodb5-replicaset-1:
    hostname: mongodb5-replicaset-1
    image: mongo:5.0
    expose:
      - 27017
    ports:
      - "27017:27017"
    restart: always
    command: mongod --port 27017 --replSet replicaSetName
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongo localhost:27017/test --quiet
      interval: 2s
      timeout: 2s
      retries: 5
      start_period: 0s
  mongodb5-replicaset-2:
    hostname: mongodb5-replicaset-2
    image: mongo:5.0
    expose:
      - 27018
    ports:
      - "27018:27018"
    restart: always
    command: mongod --port 27018 --replSet replicaSetName
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongo localhost:27018/test --quiet
      interval: 2s
      timeout: 2s
      retries: 5
      start_period: 0s
  mongodb5-replicaset-3:
    hostname: mongodb5-replicaset-3
    image: mongo:5.0
    expose:
      - 27019
    ports:
      - "27019:27019"
    restart: always
    command: mongod --port 27019 --replSet replicaSetName
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongo localhost:27019/test --quiet
      interval: 2s
      timeout: 2s
      retries: 5
      start_period: 0s

  init-mongodb5-replicaset:
    image: mongo:5.0
    restart: "no"
    depends_on:
      mongodb5-replicaset-1:
          condition: service_healthy
      mongodb5-replicaset-2:
          condition: service_healthy
      mongodb5-replicaset-3:
          condition: service_healthy
    command: >
      mongo --host mongodb5-replicaset-1:27017 --eval '
        db = (new Mongo("mongodb5-replicaset-1:27017")).getDB("test");
        config = {
            "_id" : "replicaSetName",
            "members" : [
                {
                "_id" : 0,
                "host" : "mongodb5-replicaset-1:27017"
                },
                {
                "_id" : 1,
                "host" : "mongodb5-replicaset-2:27018"
                },
                {
                "_id" : 2,
                "host" : "mongodb5-replicaset-3:27019"
                }
            ]
        };
        rs.initiate(config);
      '
```

To be able to connect to all members of replica set from your local machine, add these lines to `/etc/hosts`:
```
127.0.0.1   mongodb5-replicaset-1
127.0.0.1   mongodb5-replicaset-2
127.0.0.1   mongodb5-replicaset-3
```

After running `docker-compose up` on the yaml above, you should be able to connect to replica set MongoDB with [mongod instances](https://www.mongodb.com/docs/manual/reference/connection-string/#examples) uri:

```
mongodb://mongodb5-replicaset-1:27017,mongodb5-replicaset-2:27018,mongodb5-replicaset-3:27019/?replicaSet=replicaSetName
```