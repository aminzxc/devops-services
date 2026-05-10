### docker compose nexus proxy mirrors
```
services:
  nexus:
    image: docker.arvancloud.ir/sonatype/nexus3
    container_name: nexus
    restart: always
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms512m -Xmx1024m -XX:MaxDirectMemorySize=1024m -Djava.util.prefs.userRoot=/nexus-data/javaprefs
    volumes:
      - nexus-data:/nexus-data
    ports:
      - "8081:8081"
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
      - nexus

volumes:
  nexus-data:
    driver: local


networks:
  nexus:
    external: true

```
### set mirror for GO
```
ENV GOPROXY=http://172.24.11.152:8081/repository/go-proxy-oto/,direct
ENV GOSUMDB=off
```
### set mirror for npm
```
ENV NPM_CONFIG_REGISTRY=http://IP:8081/repository/30bime-npm-group/
```
### set mirror for node js
```
ENV npm_config_disturl=http://IP:8081/repository/30bime-nodejs-proxy/download/release
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
      <name>30bime nexus</name>
      <url>http://IP:8081/repository/30bime-maven-group/</url>
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
deb http://IP:8081/repository/30bime-ubuntu-noble/ noble main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-updates/ noble-updates main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-backports/ noble-backports main restricted universe multiverse

deb http://IP:8081/repository/30bime-ubuntu-noble-security/ noble-security main restricted universe multiverse

```
