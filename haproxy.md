### config load balancer
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        # Default SSL material locations

defaults
  default-server init-addr last,libc,none
  log global
  mode http

  option httplog
  option redispatch
  option forwardfor

  timeout connect 10s
  timeout client 60s
  timeout server 60s
  timeout tunnel 1h

frontend docker_swarm
    bind *:80
    bind *:443
    mode http
    option httplog
    default_backend swarm_managers

backend swarm_managers
    mode http
    balance roundrobin
    server master-1 192.168.20.4:80 check inter 2000 rise 2 fall 3
    server master-2 192.168.20.5:80 check inter 2000 rise 2 fall 3
    server master-3 192.168.20.6:80 check inter 2000 rise 2 fall 3


listen stats
 bind *:1936
 mode http
 option forwardfor
 option httpclose
 stats enable
 stats uri /
 stats refresh 5s
 stats show-legends
 stats realm Haproxy\ Statistics
```
### config for kubernetes
```
global
    log /dev/log local0
    log /dev/log local1 notice

    chroot /var/lib/haproxy

    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s

    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode tcp

    option tcplog
    option dontlognull

    timeout connect 5s
    timeout client  1h
    timeout server  1h
    timeout check   5s

frontend kubernetes_api_frontend
    description Kubernetes API frontend
    bind *:6443
    mode tcp

    default_backend kubernetes_api_backend

backend kubernetes_api_backend
    description Kubernetes control-plane nodes
    mode tcp

    balance roundrobin
    option tcp-check

    default-server inter 2s fall 3 rise 2

    server master1 172.24.11.16:6443 check
    server master2 172.24.11.17:6443 check
    server master3 172.24.11.15:6443 check

listen stats
 bind *:1936
 mode http
 option forwardfor
 option httpclose
 stats enable
 stats uri /
 stats refresh 5s
 stats show-legends
 stats realm Haproxy\ Statistics

```
### verify  config
```
haproxy -c -f /etc/haproxy/haproxy.cfg
```
