### docker compose nexus proxy mirrors
```
services:
  nexus:
    image: sonatype/nexus3:3.92.2
    container_name: nexus
    restart: always
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms512m -Xmx1024m -XX:MaxDirectMemorySize=1024m -Djava.util.prefs.userRoot=/nexus-data/javaprefs
    volumes:
      - nexus-data:/nexus-data
    ports:
      - "8092:8081"
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8081/service/rest/v1/status || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 10
      start_period: 180s
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    networks:
      - net
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=net"
      - "traefik.http.routers.nexus.rule=Host(`mirror.service.local`)"
      - "traefik.http.routers.nexus.entrypoints=http"
      - "traefik.http.services.nexus.loadbalancer.server.port=8081"
volumes:
  nexus-data:
    driver: local
networks:
  net:
    external: true
```
### set mirror for GO
```
ENV GOPROXY=http://IP-NEXUS:8081/repository/go-proxy-/
ENV GOSUMDB=off
Remote storage: https://proxy.golang.org

Name: go-sum-proxy
Remote storage: https://sum.golang.org
Strict Content Type Validation: OFF
ENV GOSUMDB="sum.golang.org https://nexus.example.com/repository/go-sum-proxy/"
```
### set mirror for npm
```
ENV NPM_CONFIG_REGISTRY=http://IP:8081/repository/npm-proxy/
Remote storage: https://registry.npmjs.org
```
### set mirror for node js
```
raw (proxy)
ENV npm_config_disturl=http://IP:8081/repository/nodejs-proxy/download/release
Remote storage: https://nodejs.org/
```
### set mirror for java
```
vim settings.xml

<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>2wsx@WSX</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <name>nexus</name>
      <url>http://IP:8081/repository/maven-proxy/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>


```
### set mirror alpine
```
RUN echo "http://172.24.11.152:8081/repository/oto-alpine-proxy/v3.18/main" > /etc/apk/repositories && \
    echo "http://172.24.11.152:8081/repository/oto-alpine-proxy/v3.18/community" >> /etc/apk/repositories
```
### set mirror apt
```
vim /etc/apt/sources.list
deb http://IP:8081/repository/30bime-ubuntu-noble/ noble main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-updates/ noble-updates main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-backports/ noble-backports main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-security/ noble-security main restricted universe multiverse

```
### The `RAW` repository on Nexus is used to store binary files and assets that do not have a specific format
```
Helm charts
.tar.gz, .zip, .exe, .bin files
ISOs and operating system images
Backup and archive files
PDFs, images, videos
Documentation files
Shell scripts (.sh)
Configuration files (.conf, .yaml, .json)
Ansible playbooks and roles
Terraform modules
```
