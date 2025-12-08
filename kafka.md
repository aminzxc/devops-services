### Transport
```
Kafka on each "Listener" can be in one of these states:
PLAINTEXT: Without encryption and without authentication.
SSL: Only TLS (encryption). It can be with or without mTLS (client certificate verification).
SASL_PLAINTEXT: SASL authentication without TLS (not recommended on the internet).
SASL_SSL: SASL authentication over TLS (recommended state for production).
```
### Authentication Mechanisms (SASL Mechanisms)
```
SASL/PLAIN (Simple)
Where is it stored? On each broker, in the JAAS of the same Listener.
How? With keys like user_amin="YourPass123".
Advantage/Disadvantage: Quick and simple setup; but user management is repetitive across all brokers and harder to maintain, and it's not secure without TLS.
SASL/SCRAM (SHA-256 or SHA-512)
Where is it stored? In the cluster metadata (Kafka itself). No need to define it on each broker individually.
Advantage: Centralized management, stable, more secure than PLAIN (hashed password).
```
### Permissions and Access Control (Authorization / ACLs)
```
Relationship between SASL and ACL
When you define a user on the broker (for example, with SASL/PLAIN or SCRAM), it only performs authentication.
That is, Kafka understands that "this user is real".
But Kafka still doesn't know what it can do.
Here, ACL (Access Control List) comes into play.

ACLs specify:
Which user (User)
On which resource (Resource)
=> TOPIC, GROUP, CLUSTER, TRANSACTIONAL_ID, DELEGATION_TOKEN …
What operation (Operation)
=> READ, WRITE, CREATE, DELETE, ALTER, DESCRIBE, ALTER_CONFIGS, DESCRIBE_CONFIGS, CLUSTER_ACTION, IDEMPOTENT_WRITE, …
Whether allowed or not
=> --allow OR --deny
ACLs are not applied to super users
```
### Creating a configuration file for user-side connection
```
vim /tmp/client.properties
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="oto" password="rWF696N)bMJM";

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="amin" password="rWF696N)bMJM";
```
### User-side topic creation test with authentication
```
docker run --rm -v /tmp/client.properties:/etc/kafka/client.properties \
                                           confluentinc/cp-kafka:7.5.0 \
                                           kafka-topics --bootstrap-server 79.127.125.189:9092 \
                                           --command-config /etc/kafka/client.properties \
                                           --create \
                                           --topic oto \
                                           --partitions 3 \
                                           --replication-factor 3
```
### SCRAM stands for "Proof of Password Knowledge Without Saying It"
```
In this method, the raw password is never sent.
The server already has this information:
a username (USER),
a salt (random salt),
and a complex hash of the password (not the password itself).

When the client wants to communicate with the server, it receives a salt and hash from the server and generates a hash with its own password and sends it to the server, which is called client proof. 
Because the server on its side also has this information, in the end both arrive at the same hash, which proves that neither the server is fake nor the user.
```
### create user
```
docker exec -it kafka1 kafka-configs \
                        --bootstrap-server kafka1:29092 \
                        --alter \
                        --add-config 'SCRAM-SHA-256=[iterations=8192,password=rWF696N)bMJM]' \
                        --entity-type users \
                        --entity-name amin

```
### config file kafka
```
KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required;
};
```
### docker compose SALS_SCRAM
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka1:29092,CONTROLLER://kafka1:9093,CLIENT://0.0.0.0:9092"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka1:29092,CLIENT://79.127.125.189:9092"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      # KRaft
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      # config
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      # JMX
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      # SASL/SCRAM
      KAFKA_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_SUPER_USERS: "User:oto"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
    volumes:
      - kafka1-data:/var/lib/kafka/data
      - ./kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf:ro
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka2:29093,CONTROLLER://kafka2:9193,CLIENT://0.0.0.0:9093"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka2:29093,CLIENT://79.127.125.189:9093"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9102
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_SUPER_USERS: "User:oto"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
    volumes:
      - kafka2-data:/var/lib/kafka/data
      - ./kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf:ro
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka3:29094,CONTROLLER://kafka3:9293,CLIENT://0.0.0.0:9094"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka3:29094,CLIENT://79.127.125.189:9094"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9103
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "SCRAM-SHA-256"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_SUPER_USERS: "User:oto"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
    volumes:
      - kafka3-data:/var/lib/kafka/data
      - ./kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf:ro
    networks:
      - kafka-network

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    ports:
      - "8090:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: kraft-cluster
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:29092,kafka2:29093,kafka3:29094
      KAFKA_CLUSTERS_0_METRICS_PORT: 9101
      DYNAMIC_CONFIG_ENABLED: "true"
      KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL: PLAINTEXT
    depends_on:
      - kafka1
      - kafka2
      - kafka3
    networks:
      - kafka-network

volumes:
  kafka1-data:
  kafka2-data:
  kafka3-data:

networks:
  kafka-network:
    driver: bridge

```
### docker compose SASL_PLAINTEXT
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka1:29092,CONTROLLER://kafka1:9093,CLIENT://0.0.0.0:9092"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka1:29092,CLIENT://IP:9092"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      # KRaft
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      # config
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      # JMX
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      # SASL/PLAIN
      KAFKA_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_PLAIN_SASL_JAAS_CONFIG: >
        org.apache.kafka.common.security.plain.PlainLoginModule required
        user_oto="rWF696N)bMJM";
      KAFKA_SUPER_USERS: "User:admin"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka2:29093,CONTROLLER://kafka2:9193,CLIENT://0.0.0.0:9093"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka2:29093,CLIENT://IP:9093"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9102
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_PLAIN_SASL_JAAS_CONFIG: >
        org.apache.kafka.common.security.plain.PlainLoginModule required
        user_oto="rWF696N)bMJM";
      KAFKA_SUPER_USERS: "User:admin"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,CLIENT:SASL_PLAINTEXT"
      KAFKA_LISTENERS: "INTERNAL://kafka3:29094,CONTROLLER://kafka3:9293,CLIENT://0.0.0.0:9094"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka3:29094,CLIENT://IP:9094"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9193,3@kafka3:9293"
      KAFKA_PROCESS_ROLES: "broker,controller"
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9103
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_SASL_ENABLED_MECHANISMS: "PLAIN"
      KAFKA_LISTENER_NAME_CLIENT_PLAIN_SASL_JAAS_CONFIG: >
        org.apache.kafka.common.security.plain.PlainLoginModule required
        user_oto="rWF696N)bMJM";
      KAFKA_SUPER_USERS: "User:admin"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"
    volumes:
      - kafka3-data:/var/lib/kafka/data
    networks:
      - kafka-network

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    ports:
      - "8090:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: kraft-cluster
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:29092,kafka2:29093,kafka3:29094
      KAFKA_CLUSTERS_0_METRICS_PORT: 9101
      DYNAMIC_CONFIG_ENABLED: "true"
      KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL: PLAINTEXT

    depends_on:
      - kafka1
      - kafka2
      - kafka3
    networks:
      - kafka-network

volumes:
  kafka1-data:
  kafka2-data:
  kafka3-data:

networks:
  kafka-network:
    driver: bridge
```
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
### Broker config
```
Sometimes the configurations passed from Compose to Kafka are not written directly to the container's configuration file when it runs. In this case, we must first check the variables set on the container and, after the configuration variable exists, directly check it on the broker with the apply command.
In Kafka, the first execution priority is always in seconds, the second in minutes, and the third in seconds.
The first configuration priority is on the topic itself and if not set, it will be applied to the broker configurations.
kafka-configs --bootstrap-server kafka2:29093 \
  --describe --entity-type topics --entity-name <topic-name>

example
KAFKA_LOG_RETENTION_MS=2592000000
env | grep KAFKA_LOG_RETENTION
kafka-configs --bootstrap-server kafka1:29092 --describe --entity-type brokers --entity-name 1 --all | grep -E 'log.retention.ms|log.retention.hours'
```
