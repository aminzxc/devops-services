### swarm kafka cluster with zookeeper
``` 
 zookeeper:
    hostname: zookeeper
    image: confluentinc/cp-zookeeper:latest
    user: "0:0"
    environment:
        - ZOOKEEPER_SERVER_ID=1
        - ZOOKEEPER_CLIENT_PORT=22181
        - ZOOKEEPER_TICK_TIME=2000
        - ZOOKEEPER_INIT_LIMIT=5
        - ZOOKEEPER_SYNC_LIMIT=2
    ports:
      - 22181:22181
    volumes:
      - type: "volume"
        source: "zookeeper"
        target: "/var/lib/zookeeper/data"
        volume:
          nocopy: true
      - type: "volume"
        source: "zookeeper-logs"
        target: "/var/lib/zookeeper/log"
        volume:
          nocopy: true
    deploy:
      replicas: 1
        #      placement:
        #        constraints:
        #          - node.labels.master.replica == 1
    networks:
      - pre


  kafka1:
    hostname: kafka1
    image: confluentinc/cp-kafka:7.2.1
    user: "0:0"
    ports:
     - 19092:9092
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:22181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092,EXTERNAL://192.168.20.4:19092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10485880
      KAFKA_REPLICA_FETCH_RESPONSE_MAX_BYTES: 10485880
      KAFKA_MESSAGE_MAX_BYTES: 10485880
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
        #      KAFKA_PROCESS_ROLES: broker
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 1
    networks:
      - pre

  kafka2:
    hostname: kafka2
    image: confluentinc/cp-kafka:7.2.1
    user: "0:0"
    ports:
     - 29092:9092
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:22181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9092,EXTERNAL://192.168.20.5:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10485880
      KAFKA_REPLICA_FETCH_RESPONSE_MAX_BYTES: 10485880
      KAFKA_MESSAGE_MAX_BYTES: 10485880
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
        #      KAFKA_PROCESS_ROLES: broker
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 2
    networks:
      - pre

  kafka3:
    hostname: kafka3
    image: confluentinc/cp-kafka:7.2.1
    user: "0:0"
    ports:
     - 39092:9092
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:22181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9092,EXTERNAL://192.168.20.6:39092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10485880
      KAFKA_REPLICA_FETCH_RESPONSE_MAX_BYTES: 10485880
      KAFKA_MESSAGE_MAX_BYTES: 10485880
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
        #      KAFKA_PROCESS_ROLES: broker
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 3
    networks:
      - pre

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - target: 8080
        published: 8090
        protocol: tcp
        mode: ingress
    environment:
      KAFKA_CLUSTERS_0_NAME: kraft-cluster
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:9092,kafka2:9092,kafka3:9092
      KAFKA_CLUSTERS_0_METRICS_PORT: 9101
      DYNAMIC_CONFIG_ENABLED: 'true'
    networks:
      - pre
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.kafka-ui.rule=Host(`kafkak.otomob.ir`)"
        - "traefik.http.routers.kafka-ui.entrypoints=http"
        - "traefik.http.services.kafka-ui.loadbalancer.server.port=8080"
        - "traefik.docker.network=pre"

          #        - "traefik.http.middlewares.kafka-ui-auth.basicauth.removeheader=true"
          #    - "traefik.http.middlewares.kafka-ui-auth.basicauth.realm=KafkaUI"
          #     - "traefik.http.middlewares.kafka-ui-auth.basicauth.users=amin:$$2y$$05$$wkAHs8nC0N4KAJt5EbuQGe76QhrGWoPW4wyR0a6vhLvKprkVFF5cy"
          #    - "traefik.http.routers.kafka-ui.middlewares=kafka-ui-auth@docker"


      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```
### swarm kafka cluster with Kraft
```
  kafka1:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka1
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka1:29092,PLAINTEXT_HOST://kafka1:9092'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka1:29092,CONTROLLER://kafka1:9093,PLAINTEXT_HOST://0.0.0.0:9092'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/var/lib/kafka/data'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: kafka1
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    networks:
      - pre
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 1
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 6G
        reservations:
          memory: 1G
    volumes:
      - kafka1:/var/lib/kafka/data
  kafka2:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka2
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka2:29093,PLAINTEXT_HOST://kafka2:9093'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka2:29093,CONTROLLER://kafka2:9193,PLAINTEXT_HOST://0.0.0.0:9093'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/var/lib/kafka/data'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9102
      KAFKA_JMX_HOSTNAME: kafka2
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    networks:
      - pre
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 2
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 6G
        reservations:
          memory: 1G
    volumes:
      - kafka2:/var/lib/kafka/data

  kafka3:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka3
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka3:29094,PLAINTEXT_HOST://kafka3:9094'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka3:29094,CONTROLLER://kafka3:9293,PLAINTEXT_HOST://0.0.0.0:9094'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/var/lib/kafka/data'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9103
      KAFKA_JMX_HOSTNAME: kafka3
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    networks:
      - pre
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 6G
        reservations:
          memory: 1G
    volumes:
      - kafka3:/var/lib/kafka/data



```
### kafka UI
```
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - target: 8080
        published: 8090
        protocol: tcp
        mode: ingress
    environment:
      KAFKA_CLUSTERS_0_NAME: kraft-cluster
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:29092,kafka2:29093,kafka3:29094
      KAFKA_CLUSTERS_0_METRICS_PORT: 9101
      DYNAMIC_CONFIG_ENABLED: 'true'
      AUTH_TYPE: "LOGIN_FORM"
      SPRING_SECURITY_USER_NAME: "user"
      SPRING_SECURITY_USER_PASSWORD: "kaa#stlknefdalage*TO"
    networks:
      - pre
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.kafka-ui.rule=Host(`host.ir`)"
        - "traefik.http.routers.kafka-ui.entrypoints=http"
        - "traefik.http.services.kafka-ui.loadbalancer.server.port=8080"
        - "traefik.docker.network=pre"
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```
### kafka cluster with zookeper cluster
```
  zookeeper1:
    hostname: zookeeper1
    image: confluentinc/cp-zookeeper:latest
    user: "0:0"
    environment:
        - ZOOKEEPER_SERVER_ID=1
        - ZOOKEEPER_CLIENT_PORT=22181
        - ZOOKEEPER_TICK_TIME=2000
        - ZOOKEEPER_INIT_LIMIT=5
        - ZOOKEEPER_SYNC_LIMIT=2
        - ZOOKEEPER_SERVERS=zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - "22181:22181"
      - "22888:22888"
      - "23888:23888"
    volumes:
      - type: "volume"
        source: "zookeeper1"
        target: "/var/lib/zookeeper/data"
        volume:
          nocopy: true
      - type: "volume"
        source: "zookeeper1-logs"
        target: "/var/lib/zookeeper/log"
        volume:
          nocopy: true
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 1
    networks:
      - stage

  zookeeper2:
    hostname: zookeeper2
    image: confluentinc/cp-zookeeper:latest
    user: "0:0"
    environment:
        - ZOOKEEPER_SERVER_ID=2
        - ZOOKEEPER_CLIENT_PORT=32181
        - ZOOKEEPER_TICK_TIME=2000
        - ZOOKEEPER_INIT_LIMIT=5
        - ZOOKEEPER_SYNC_LIMIT=2
        - ZOOKEEPER_SERVERS=zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - "32181:32181"
      - "32888:32888"
      - "33888:33888"
    volumes:
      - type: "volume"
        source: "zookeeper2"
        target: "/var/lib/zookeeper/data"
        volume:
          nocopy: true
      - type: "volume"
        source: "zookeeper2-logs"
        target: "/var/lib/zookeeper/log"
        volume:
          nocopy: true
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 2
    networks:
      - stage

  zookeeper3:
    hostname: zookeeper3
    image: confluentinc/cp-zookeeper:latest
    user: "0:0"
    environment:
        - ZOOKEEPER_SERVER_ID=3
        - ZOOKEEPER_CLIENT_PORT=42181
        - ZOOKEEPER_TICK_TIME=2000
        - ZOOKEEPER_INIT_LIMIT=5
        - ZOOKEEPER_SYNC_LIMIT=2
        - ZOOKEEPER_SERVERS=zookeeper1:22888:23888;zookeeper2:32888:33888;zookeeper3:42888:43888
    ports:
      - "42181:42181"
      - "42888:42888"
      - "43888:43888"
    volumes:
      - type: "volume"
        source: "zookeeper3"
        target: "/var/lib/zookeeper/data"
        volume:
          nocopy: true
      - type: "volume"
        source: "zookeeper3-logs"
        target: "/var/lib/zookeeper/log"
        volume:
          nocopy: true
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 3
    networks:
      - stage

  kafka1:
    hostname: kafka1
    image: confluentinc/cp-kafka:latest
    user: "0:0"
    ports:
     - 19092:19092
    volumes:
      - type: "volume"
        source: "kafka1"
        target: "/var/lib/kafka/data"
        volume:
          nocopy: true
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 192.168.20.4:22181,192.168.20.5:32181,192.168.20.6:42181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.20.4:19092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
      KAFKA_DELETE_TOPIC_ENABLE: "true"
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 1
    networks:
      - stage

  kafka2:
    hostname: kafka2
    image: confluentinc/cp-kafka:latest
    user: "0:0"
    ports:
     - 29092:29092
    volumes:
      - type: "volume"
        source: "kafka2"
        target: "/var/lib/kafka/data"
        volume:
          nocopy: true
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: 192.168.20.4:22181,192.168.20.5:32181,192.168.20.6:42181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.20.5:29092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
      KAFKA_DELETE_TOPIC_ENABLE: "true"
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 2
    networks:
      - stage

  kafka3:
    hostname: kafka3
    image: confluentinc/cp-kafka:latest
    user: "0:0"
    ports:
     - 39092:39092
    volumes:
      - type: "volume"
        source: "kafka3"
        target: "/var/lib/kafka/data"
        volume:
          nocopy: true
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: 192.168.20.4:22181,192.168.20.5:32181,192.168.20.6:42181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.20.6:39092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
      KAFKA_DELETE_TOPIC_ENABLE: "true"
    deploy:
      placement:
        constraints:
          - node.labels.master.replica == 3
    networks:
      - stage
```
### kafka cluster docker compose
```
services:
  kafka1:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka1
    container_name: kafka1
    ports:
      - "9092:9092"
      - "9101:9101"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka1:29092,PLAINTEXT_HOST://localhost:9092'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka1:29092,CONTROLLER://kafka1:9093,PLAINTEXT_HOST://0.0.0.0:9092'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    volumes:
      - kafka1-data:/var/lib/kafka/data
    networks:
      - kafka-network

  kafka2:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka2
    container_name: kafka2
    ports:
      - "9093:9093"
      - "9102:9102"
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka2:29093,PLAINTEXT_HOST://localhost:9093'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka2:29093,CONTROLLER://kafka2:9193,PLAINTEXT_HOST://0.0.0.0:9093'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9102
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    volumes:
      - kafka2-data:/var/lib/kafka/data
    networks:
      - kafka-network

  kafka3:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka3
    container_name: kafka3
    ports:
      - "9094:9094"
      - "9103:9103"
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka3:29094,PLAINTEXT_HOST://localhost:9094'
      KAFKA_LISTENERS: 'PLAINTEXT://kafka3:29094,CONTROLLER://kafka3:9293,PLAINTEXT_HOST://0.0.0.0:9094'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka1:9093,2@kafka2:9193,3@kafka3:9293'
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9103
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qg'
    volumes:
      - kafka3-data:/var/lib/kafka/data
    networks:
      - kafka-network
```
### test cluster
```
kafka-metadata-quorum --bootstrap-server kafka1:29092 describe --status
kafka-metadata-quorum --bootstrap-server kafka1:29092 describe --replication
```
