### `etcd` clustering & apisix
```
version: '3.8'
services:
  etcd-node-0:
    image: bitnami/etcd:3.4.15
    hostname: etcd-node-0
    ports:
      - "2379:2379"
      - "2380:2380"
    volumes:
      - etcd-0:/bitnami/etcd
    networks:
      - stage
    environment:
      - ETCD_LOGGER=zap
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd-node-0
      - ETCD_DATA_DIR=/bitnami/etcd/data
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd-node-0:2380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd-node-0:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_INITIAL_CLUSTER=etcd-node-0=http://etcd-node-0:2380,etcd-node-1=http://etcd-node-1:2380,etcd-node-2=http://etcd-node-2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 1

  etcd-node-1:
    image: bitnami/etcd:3.4.15
    hostname: etcd-node-1
    ports:
      - "2479:2379"
      - "2480:2380"
    volumes:
      - etcd-1:/bitnami/etcd
    networks:
      - stage
    environment:
      - ETCD_LOGGER=zap
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd-node-1
      - ETCD_DATA_DIR=/bitnami/etcd/data
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd-node-1:2380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd-node-1:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_INITIAL_CLUSTER=etcd-node-0=http://etcd-node-0:2380,etcd-node-1=http://etcd-node-1:2380,etcd-node-2=http://etcd-node-2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 2

  etcd-node-2:
    image: bitnami/etcd:3.4.15
    hostname: etcd-node-2
    ports:
      - "2579:2379"
      - "2580:2380"
    volumes:
      - etcd-2:/bitnami/etcd
    networks:
      - stage
    environment:
      - ETCD_LOGGER=zap
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd-node-2
      - ETCD_DATA_DIR=/bitnami/etcd/data
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://etcd-node-2:2380
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd-node-2:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_INITIAL_CLUSTER=etcd-node-0=http://etcd-node-0:2380,etcd-node-1=http://etcd-node-1:2380,etcd-node-2=http://etcd-node-2:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.master.replica == 3


  apisix:
    image: apache/apisix:3.11.0-debian
    configs:
      - source: apisix_config
        target: /usr/local/apisix/conf/config.yaml
    depends_on:
      - etcd-node-1
      - etcd-node-0
      - etcd-node-2
    ports:
      - target: 80
        published: 80
        protocol: tcp
        mode: host
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
        delay: 5s
        max_attempts: 3
        window: 120s
    networks:
      - stage


  apisix-dashboard:
    image: apache/apisix-dashboard:3.0.1-alpine
    configs:  # Use Docker configs instead of bind mounts
      - source: dashboard_config
        target: /usr/local/apisix-dashboard/conf/conf.yaml
    depends_on:
      - etcd-node-1
      - etcd-node-0
      - etcd-node-2
    ports:
      - target: 9000
        published: 9000
        protocol: tcp
        mode: host
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: any
        delay: 5s
        max_attempts: 3
        window: 120s
    networks:
      - stage
configs:
  dashboard_config:
    file: ./dashboard_conf/conf.yaml
  apisix_config:
    file: ./apisix_conf/config.yaml

```
### download  & install`ETCDCTL`
```
https://github.com/etcd-io/etcd/releases/download/v3.5.15/etcd-v3.5.15-linux-amd64.tar.gz
tar -xzvf etcd-v3.5.15-linux-amd64.tar.gz
install etcdctl /usr/local/bin/
```
### backup etcd
```
etcdctl snapshot save /opt/etcd-snapshot.db
```
### backuo etcd with tls
```
etcdctl snapshot save backup-etcd.db --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt
```
### status backup
```
etcdctl snapshot status /opt/etcd-snapshot.db -w table
```
### restore backup
etcdctl snapshot restore snapshot20240718.db \
  --data-dir /var/lib/etcd-from-backup \ # creates the new path himself
  --initial-cluster master1=https://192.168.1.5:2380 \
  --initial-advertise-peer-urls https://192.168.1.5:2380 \
  --name master1

```
