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
