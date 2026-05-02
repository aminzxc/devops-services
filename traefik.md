### config for swarm
```
  traefik:
    image: traefik:v2.11
    command:
      - "--log.level=INFO"
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.http.address=:80"
      - "--api=true"
      - "--ping=true"
      - "--accesslog=true"
      - "--accesslog.bufferingsize=100"
      - "--api.insecure=true"
      - "--providers.docker.network=pre"
      - "--entrypoints.http.forwardedHeaders.trustedIPs=192.168.20.3"
      - "--log.format=json"
      - "--accesslog.format=json"

      - "--metrics.prometheus=true"
      - "--metrics.prometheus.entryPoint=metrics"
      - "--metrics.prometheus.addEntryPointsLabels=true"
      - "--metrics.prometheus.addServicesLabels=true"
      - "--metrics.prometheus.buckets=0.05, 0.1, 0.3, 0.6, 1, 3, 5"
      - "--entrypoints.metrics.address=:8082"
    ports:
      - target: 80
        published: 80
        protocol: tcp
        mode: host
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/etc/localtime:/etc/localtime:ro"
    deploy:
      mode: global
      placement:
        constraints:
          - node.role==manager
            #      replicas: 3
    networks:
      - pre
```
### config for compose
```
  traefik:
    image: traefik:v2.11.31
    container_name: traefik
    restart: unless-stopped
    command:
      - "--log.level=INFO"
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
      - "--api.insecure=false"
      - "--entrypoints.traefik.address=:8080"
      - "--ping.entrypoint=traefik"
      - "--accesslog.format=json"
      - "--providers.docker.network=oto-network"
    ports:
      - "80:80"
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/ping"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 20s
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.oto.ir`)"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=wimdfgnd:$$2y$$05$$3DCy/GnLngnxO3dJlaSNhO6BQ2h7Iv9ejevasneaXiob2x9nhN8TW"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - oto-network

```
### traefik for local dns
```
  traefik:
    image: traefik:v2.11.31
    container_name: traefik
    restart: always
    command:
      - "--log.level=INFO"
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
      - "--entrypoints.traefik.address=:8081"
      - "--ping.entrypoint=traefik"
      - "--accesslog.format=json"
      - "--providers.docker.network=dev-network"
    ports:
      - "80:80"
      - "8081:8081"
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8081/ping"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 20s
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.30bime.ir`)"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=wimdfgnd:$$2y$$05$$3DCy/GnLngnxO3dJlaSNhO6BQ2h7Iv9ejevasneaXiob2x9nhN8TW"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - dev-network
```
### config for path prefix
```
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=dev-network"
      - "traefik.http.routers.insurance.rule=Host(`30bime.ir`) && PathPrefix(`/insurance-api`)"
      - "traefik.http.routers.insurance.entrypoints=http"
      - "traefik.http.middlewares.insurance-strip.stripprefix.prefixes=/insurance-api"
      - "traefik.http.routers.insurance.middlewares=insurance-strip"
      - "traefik.http.services.insurance.loadbalancer.server.port=8200"
```
### config for SSL TLS
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout mattermost.key \
-out mattermost.crt \
-subj "/CN=mattermost.local" \
-addext "subjectAltName=DNS:mattermost.local"

```
```
  traefik:
    image: traefik:v2.11.31
    container_name: traefik
    restart: always
    command:
      - "--log.level=INFO"
      - "--providers.docker=true"
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=false"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.http.address=:80"
      - "--entrypoints.https.address=:443"
      - "--api=true"
      - "--ping=true"
      - "--accesslog=true"
      - "--accesslog.bufferingsize=100"
      - "--api.insecure=true"
      - "--entrypoints.traefik.address=:8081"
      - "--ping.entrypoint=traefik"
      - "--accesslog.format=json"
      - "--providers.docker.network=matter-net"
      - "--providers.file.directory=/etc/traefik/dynamic"
      - "--providers.file.watch=true"
    ports:
      - "80:80"
      - "443:443"
      - "8081:8081"
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8081/ping"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 20s
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.30bime.ir`)"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=wimdfgnd:$$2y$$05$$3DCy/GnLngnxO3dJlaSNhO6BQ2h7Iv9ejevasneaXiob2x9nhN8TW"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "./traefik/certs:/etc/traefik/certs:ro"
      - "./traefik/dynamic:/etc/traefik/dynamic:ro"
    networks:
      - matter-net
------>
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mattermost.rule=Host(`mattermost.local`)"
      - "traefik.http.routers.mattermost.entrypoints=https"
      - "traefik.http.routers.mattermost.tls=true"
      - "traefik.http.services.mattermost.loadbalancer.server.port=8065"
```
```
mkdir -p ./traefik/dynamic
certs.yml
tls:
  certificates:
    - certFile: /etc/traefik/certs/mattermost.crt
      keyFile: /etc/traefik/certs/mattermost.key
-----------------------------------------------------------------
mkdir -p ./traefik/dynamic
mv mattermost.crt  mattermost.key .
```
### Trusting a cert in the browser
```
import sample.crt
chrome://certificate-manager/localcerts/usercerts
restart chrome
```
