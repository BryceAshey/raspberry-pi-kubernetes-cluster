# Master Node Setup

## Configure Keepalive
We use keepalive on each of the 3 master nodes to ensure a single static IP is available at all times for the Kubernetes API. If a node fails then keepalive will move the IP to one of the remaining "up" nodes. As with before change the IPs found within to meet your needs.

Install Keepalive
```
> sudo apt-get install keepalived
```

Configure Keepalive

Note: on each master node you need to change the "unicast_src_ip" to match the IP of the node. Then change the "unicast_peer_ip" to match the other two nodes.
```
> sudo nano /etc/keepalived/keepalived.conf

# replace with the following:
vrrp_script haproxy-check {
    script "killall -0 haproxy"
    interval 2
    weight 20
}
 
vrrp_instance haproxy-vip {
    state BACKUP
    priority 101
    interface eth0
    virtual_router_id 47
    advert_int 3
 
    unicast_src_ip 192.168.200.252
    unicast_peer {
        192.168.200.250
        192.168.200.251
    }
 
    virtual_ipaddress {
        192.168.200.249 
    }
 
    track_script {
        haproxy-check weight 20
    }
}

```

Reboot
```
> sudo reboot now
```

## HAProxy Setup
We use HAProxy to route traffic from the master node IP to the aforementioned keepalive IP (in this example 192.168.200.249).

Install HAProxy
```
> sudo apt-get install haproxy
```

Enable on Startup
```
> sudo nano /etc/default/haproxy
# Set	ENABLED=1
```
Configure HAProxy
The SSH proxy is optional - I found that sometimes HAProxy would interfere with SSH unless I put the proxy in. Something to figure out longer term.
```
> sudo nano /etc/haproxy/haproxy.cfg

# Edit with the following

global
        log 127.0.0.1:514  local0
        log 127.0.0.1:514  local0 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd lis$
        stats timeout 30s
#       user haproxy
#       group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL). This list is from:
        #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
        # An alternative list with additional directives can be obtained from
        #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?serv$
        ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:EC$
        ssl-default-bind-options no-sslv3

defaults
        log     global
        mode    tcp
        option  tcplog
        option  dontlognull
        timeout connect 1s
        timeout client  20s
        timeout server  20s
        timeout client-fin 20s
        timeout tunnel 1h
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen stats
  bind    *:9000
  mode    http
  stats   enable
  stats   hide-version
  stats   uri       /stats
  stats   refresh   30s
  stats   realm     Haproxy\ Statistics
  stats   auth      Admin:Password

frontend k8s-api
        bind 192.168.200.249:443
        bind 127.0.0.1:443
        mode tcp
        option tcplog
        default_backend k8s-api

frontend ssh-proxy
        bind 192.168.200.251:2222
        mode tcp
        option tcplog
        default_backend ssh-proxy-backend

backend k8s-api
        mode tcp
        server kuber04m01 192.168.200.250:6443 check
        server kuber04m02 192.168.200.251:6443 check
        server kuber04m03 192.168.200.252:6443 check

backend ssh-proxy-backend
        mode tcp
        server kuber04m02 192.168.200.251:22 check id 1
```

Reboot
```
> sudo reboot now
```

