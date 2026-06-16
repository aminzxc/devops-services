### docker compose operational
```
services:
  mattermost-postgres:
    image: postgres:16-alpine
    container_name: mattermost-postgres
    restart: always
    environment:
      - POSTGRES_USER=mattermost
      - POSTGRES_PASSWORD=mattermost
      - POSTGRES_DB=mattermost
    ports:
      - "5433:5432"
    volumes:
      - mattermost-db-data:/var/lib/postgresql/data
    networks:
      - matter-net

  mattermost:
    image: mattermost/mattermost-team-edition:11.3
    container_name: mattermost
    restart: always
    depends_on:
      - mattermost-postgres
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://mattermost:mattermost@mattermost-postgres:5432/mattermost?sslmode=disable&connect_timeout=10
      - MM_SERVICESETTINGS_SITEURL=http://172.24.11.168:8065
      - MM_SERVICESETTINGS_ENABLEINCOMINGWEBHOOKS=true
      - MM_SERVICESETTINGS_ENABLEOUTGOINGWEBHOOKS=true
    ports:
      - "8065:8065"
      - "8443:8443/tcp"
      - "8443:8443/udp"
    volumes:
      - mattermost-config:/mattermost/config
      - mattermost-data:/mattermost/data
      - mattermost-logs:/mattermost/logs
      - mattermost-plugins:/mattermost/plugins
    networks:
      - matter-net
networks:
  matter-net:
    external: true
volumes:
  mattermost-db-data:
    name: mattermost_mattermost-db-data
  mattermost-config:
    name: mattermost_mattermost-config
  mattermost-data:
    name: mattermost_mattermost-data
  mattermost-logs:
    name: mattermost_mattermost-logs
  mattermost-plugins:
    name: mattermost_mattermost-plugins
```
### To enable the call feature, the module must be installed from the market place. In the call configuration, test mode must be turned off. The ICE Servers Configurations value must be empty
### Configure Chrome browser to use call
```
chrome://flags/#unsafely-treat-insecure-origin-as-secure
http://IP:8065
Enabled
relunch
```
### For proper operation, the firewall must be turned off
### manage by `https`
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout mattermost.key \
-out mattermost.crt \
-subj "/CN=mattermost.local" \
-addext "subjectAltName=DNS:mattermost.local"

openssl x509 -in mattermost.crt -noout -subject -issuer -dates -ext subjectAltName
```
```
services:
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

```
### label for traefik
```
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mattermost.rule=Host(`mattermost.local`)"
      - "traefik.http.routers.mattermost.entrypoints=https"
      - "traefik.http.routers.mattermost.tls=true"
      - "traefik.http.services.mattermost.loadbalancer.server.port=8065"

```
### Add *.crt to Google Chrome
```
chrome://certificate-manager/localcerts/usercerts
```
### A cert must be generated for each domain
م
مش
