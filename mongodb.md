### mongo cluster
```
  mongo-1:
    image: mongo:latest
    hostname: mongo-1
    command: mongod --replSet mydb --bind_ip_all
    ports:
      - 27017:27017
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.master.replica == 1
    volumes:
      - mongo-data-1:/data/db
      - mongo-config-1:/data/configdb
    networks:
      - stage

  mongo-2:
    image: mongo:latest
    hostname: mongo-2
    command: mongod --replSet mydb --bind_ip_all
    ports:
      - 27018:27017
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.master.replica == 2
    volumes:
      - mongo-data-2:/data/db
      - mongo-config-2:/data/configdb
    networks:
      - stage

  mongo-3:
    image: mongo:latest
    hostname: mongo-3
    command: mongod --replSet mydb --bind_ip_all
    ports:
      - 27019:27017
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.labels.master.replica == 3
    volumes:
      - mongo-data-3:/data/db
      - mongo-config-3:/data/configdb
    networks:
      - stage

```
