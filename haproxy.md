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
