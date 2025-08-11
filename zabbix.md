### config `zabbix` docker
```
services:
  mysql-server:
    image: mysql:8.0.36
    container_name: "mysql"
      #    deploy:
      #      resources:
      #        limits:
      #          cpus: '1'
      #          memory: 900M

    networks:
      - zabbix
    command:
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-authentication-plugin=mysql_native_password
    environment:
      - MYSQL_USER=zabbix
      - MYSQL_DATABASE=zabbix
      - MYSQL_PASSWORD=dp145
      - MYSQL_ROOT_PASSWORD=dp145

    volumes:
      - my-db-mysql:/var/lib/mysql
      - ./zabbix-mysql/backups:/backups
    restart: always

  zabbix-server-mysql:
    image: zabbix/zabbix-server-mysql:7.0.17-ubuntu
    hostname: zabbix-server-mysql
    container_name: "zabbix-server"
      #    deploy:
      #      resources:
      #        limits:
      #          cpus: '0.50'
      #          memory: 200M

    networks:
      - zabbix

    ports:
      - 10051:10051
        # - 10050:10050
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zabbix-data/alertscripts:/usr/lib/zabbix/alertscripts:rw
      - ./zabbix-data/externalscripts:/usr/lib/zabbix/externalscripts:rw
      - ./zabbix-data/export:/var/lib/zabbix/export:rw
      - ./zabbix-data/modules:/var/lib/zabbix/modules:ro
      - ./zabbix-data/enc:/var/lib/zabbix/enc:ro
      - ./zabbix-data/ssh_keys:/var/lib/zabbix/ssh_keys:rw
      - ./zabbix-data/mibs:/var/lib/zabbix/mibs:ro
      - ./zabbix-data/snmptraps:/var/lib/zabbix/snmptraps:rw
    environment:
      - DB_SERVER_HOST=mysql-server
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=dp145
      - MYSQL_ROOT_PASSWORD=dp145
      - ZBX_SERVER_HOST=79.127.125.189
    depends_on:
      - mysql-server
    restart: always

  zabbix-web-nginx-mysql:
    image: zabbix/zabbix-web-nginx-mysql:7.0.17-ubuntu
    container_name: "zabbix-nginx"
    networks:
      - zabbix
        #    deploy:
        #      resources:
        #        limits:
        #          cpus: '0.50'
        #          memory: 300M
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zabbix-nginx/nginx:/etc/ssl/nginx:rw
      - ./zabbix-nginx/modules/:/usr/share/zabbix/modules/:ro
    environment:
      - ZBX_SERVER_HOST=zabbix-server-mysql
      - DB_SERVER_HOST=mysql-server
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=dp145
      - MYSQL_ROOT_PASSWORD=dp145
      - PHP_TZ=Asia/Tehran
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`zabbix.otomob.ir`)"
      - "traefik.http.routers.web.entrypoints=http"
      - "traefik.http.services.web.loadbalancer.server.port=8080"
      - "traefik.docker.network=zabbix"

    depends_on:
      - mysql-server
      - zabbix-server-mysql
    restart: always

  traefik:
    image: traefik:v2.11
    command:
      - "--log.level=WARN"
      - "--providers.docker=true"
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=false"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.http.address=:80"
      - "--api=true"
      - "--ping=true"
      - "--accesslog=true"
      - "--accesslog.bufferingsize=100"
      - "--api.insecure=true"
      - "--providers.docker.network=zabbix"
    ports:
      - "80:80"
      - "8088:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - zabbix
networks:
  zabbix:
    external: true
volumes:
  my-db-mysql:

```
