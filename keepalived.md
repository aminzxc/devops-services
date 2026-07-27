### keepalived `master` `backup`
### node master
```
global_defs {
   router_id HAPROXY_PRIMARY
   enable_script_security
   script_user root
}

vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 2
    timeout 1
    fall 3
    rise 2
   }

vrrp_instance K8S_API {
   state MASTER
   interface ens160
   virtual_router_id 51
   priority 100
   advert_int 1

   unicast_src_ip 172.24.11.137
   check_unicast_src
   unicast_fault_no_peer
   track_src_ip
   unicast_peer {
       172.24.11.18
   }

   virtual_ipaddress {
      172.24.11.10/24 dev ens160
   }

   track_script {
      check_haproxy
   }
}

```
### backup node  
```
global_defs {
   router_id HAPROXY_SECONDARY
   enable_script_security
   script_user root
}

vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 2
    timeout 1
    fall 3
    rise 2
   }

vrrp_instance K8S_API {
   state BACKUP
   interface ens160
   virtual_router_id 51
   priority 90
   advert_int 1

   unicast_src_ip 172.24.11.18
   check_unicast_src
   unicast_fault_no_peer
   track_src_ip
   unicast_peer {
       172.24.11.137
   }

   virtual_ipaddress {
      172.24.11.10/24 dev ens160
   }

   track_script {
      check_haproxy
   }
}

```
### test on node master
```
ip -br -c a
```
```
tcpdump -ni ens160  'ip proto 112 and (host 172.24.11.137 or host 172.24.11.18)'
```
