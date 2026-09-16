### docker compose nexus proxy mirrors
```
services:
  nexus:
    image: sonatype/nexus3:3.96.1
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

go (proxy)
Remote storage: https://proxy.golang.org

or

Name: go-sum-proxy
Remote storage: https://sum.golang.org
Strict Content Type Validation: OFF
ENV GOSUMDB="sum.golang.org https://nexus.example.com/repository/go-sum-proxy/"
```
### set mirror for npm
```
ENV NPM_CONFIG_REGISTRY=http://IP:8081/repository/npm-proxy/

npm (proxy)
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

COPY settings.xml .
RUN mvn -q -DskipTests package -s settings.xml

maven2 (proxy)

Remote storage: https://repo1.maven.org/maven2/
```
### set mirror alpine
```
RUN echo "http://IP:8081/repository/alpine-proxy/v3.18/main" > /etc/apk/repositories && \
    echo "http://IP:8081/repository/alpine-proxy/v3.18/community" >> /etc/apk/repositories

alpine (proxy)

Remote storage: https://dl-cdn.alpinelinux.org/alpine/
```
### set mirror apt
```
ubuntu-proxy
Remote Storage:
https://archive.ubuntu.com/ubuntu/

ubuntu-security-proxy
Remote Storage:
https://security.ubuntu.com/ubuntu/

#########################################################
debian-proxy
Remote:
https://deb.debian.org/debian/

debian-security-proxy
Remote:
https://security.debian.org/debian-security/
########################################################
ARG NEXUS=http://IP:8081

RUN rm -f /etc/apt/sources.list.d/ubuntu.sources \
    && printf '%s\n' \
       "deb ${NEXUS}/repository/ubuntu-proxy/ noble main restricted universe multiverse" \
       "deb ${NEXUS}/repository/ubuntu-proxy/ noble-updates main restricted universe multiverse" \
       "deb ${NEXUS}/repository/ubuntu-proxy/ noble-backports main restricted universe multiverse" \
       "deb ${NEXUS}/repository/ubuntu-security-proxy/ noble-security main restricted universe multiverse" \
       > /etc/apt/sources.list
or
RUN rm -f /etc/apt/sources.list.d/debian.sources \
    && printf '%s\n' \
       "deb ${NEXUS}/repository/debian-proxy/ bookworm main" \
       "deb ${NEXUS}/repository/debian-proxy/ bookworm-updates main" \
       "deb ${NEXUS}/repository/debian-security-proxy/ bookworm-security main" \
       > /etc/apt/sources.list
```
### ### set mirror php
```
RUN composer config -g secure-http false && \
    composer config -g repos.packagist composer http://IP:8081/repository/php-proxy/

composer (proxy)
Remote Storage:
https://repo.packagist.org
```
### set mirror python
```
ENV PIP_INDEX_URL=http://IP:8081/repository/python-proxy/simple
ENV PIP_TRUSTED_HOST=IP:8081

pypi (proxy)
Remote Storage:
https://pypi.org/
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
