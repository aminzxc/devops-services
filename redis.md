### redis clustering `SHARDING`
```
  redis1:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis1-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    deploy:
      placement:
        constraints: [node.labels.master.replica == 1]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis2:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis2-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

    deploy:
      placement:
        constraints: [node.labels.master.replica == 2]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis3:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis3-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

    deploy:
      placement:
        constraints: [node.labels.master.replica == 3]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis4:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis4-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

    deploy:
      placement:
        constraints: [node.labels.master.replica == 1]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis5:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis5-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

    deploy:
      placement:
        constraints: [node.labels.master.replica == 2]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis6:
    image: redis:7.4.2
    command: >
      redis-server --cluster-enabled yes
                   --cluster-config-file nodes.conf
                   --cluster-node-timeout 5000
                   --appendonly yes
    volumes:
      - redis6-data:/data
    networks:
      - pre
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

    deploy:
      placement:
        constraints: [node.labels.master.replica == 3]
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
        window: 60s

  redis-insight:
    image: redislabs/redisinsight:2.70.1
    container_name: redis-insight
    networks:
      - pre
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

      restart_policy:
        condition: on-failure
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.redis-insight.rule=Host(`redis.otomob.ir`)"
        - "traefik.http.routers.redis-insight.entrypoints=http"
        - "traefik.http.services.redis-insight.loadbalancer.server.port=5540"
        - "traefik.docker.network=pre"
        - "traefik.http.middlewares.redis-auth.basicauth.users=amin:$$2y$$05$$SDK4/rPXo7yMhdEilysIiuK9JHlPJzkh/CTG/ocUrCyfDBh3a6XaG"
        - "traefik.http.middlewares.redis-auth.basicauth.removeheader=true"
        - "traefik.http.middlewares.redis-auth.basicauth.realm=RedisInsight"
        - "traefik.http.routers.redis-insight.middlewares=redis-auth@docker"
volumes:
  redis1-data:
  redis2-data:
  redis3-data:
  redis4-data:
  redis5-data:
  redis6-data:
```
### check config file
```
docker compose -f file.yml config
docker stack config -c file.yml
```
### check status cluster `redis cli` 
```
cluster nodes
cluster info
```
### Running the cluster to join nodes
```
docker exec -it redis1 bash
```
```
redis-cli --cluster create redis1:6379 redis2:6379 redis3:6379 redis4:6379 redis5:6379 redis6:6379 --cluster-replicas 1
```
### redis cludter `MASTER & SLAVE`
```
  redis-master:
    image: redis:7.4.2
    hostname: redis-master
    command: redis-server --requirepass Redisoto123
    ports:
      - 6379:6379
    networks:
      - pre
    volumes:
      - redis-master-data:/data
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 1
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "Redisoto123", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis-slave-1:
    image: redis:7.4.2
    hostname: redis-slave-1
    command: redis-server --slaveof redis-master 6379 --masterauth Redisoto123 --requirepass Redisoto123
    ports:
      - 6380:6379
    networks:
      - pre
    volumes:
      - redis-slave-1-data:/data
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 2
      restart_policy:
        condition: on-failure
    depends_on:
      - redis-master
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "Redisoto123", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis-slave-2:
    image: redis:7.4.2
    hostname: redis-slave-2
    command: redis-server --slaveof redis-master 6379 --masterauth Redisoto123 --requirepass Redisoto123
    ports:
      - 6381:6379
    networks:
      - pre
    volumes:
      - redis-slave-2-data:/data
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 3
      restart_policy:
        condition: on-failure
    depends_on:
      - redis-master
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "Redisoto", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
```
