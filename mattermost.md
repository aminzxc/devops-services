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
### label for traefik
```
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mattermost.rule=Host(`mattermost.local`)"
      - "traefik.http.routers.mattermost.entrypoints=https"
      - "traefik.http.routers.mattermost.tls=true"
      - "traefik.http.services.mattermost.loadbalancer.server.port=8065"

```
م
مش
