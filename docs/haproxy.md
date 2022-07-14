> To expose your Hyperion, History or Atomic API you need a load balancer. There are several community balancers available for use and the following ones will be explained

- HAProxy
- NGINX
- Traefik

> Other components that will be covered

- ACL
- Rate limiting with explanation and examples **(stick-tables, tarpit, etc)**
- Cors setup
- Letsencrypt to generate a public certificate

> **_NOTE:_** In each section, the input commands will be numbered, followed by an output

## HAProxy

### Install

Installation on Ubuntu (this installation was performed on ubuntu 18.04, but will be the same on 20.04)
```
1. sudo apt install --no-install-recommends software-properties-common
2. sudo add-apt-repository ppa:vbernat/haproxy-2.4 -y
```
Result =>
```
Reading package lists... Done
Building dependency tree
Reading state information... Done
software-properties-common is already the newest version (0.96.24.32.18).
The following packages were automatically installed and are no longer required:
  liblua5.3-0 linux-image-4.15.0-156-generic linux-modules-4.15.0-156-generic linux-modules-extra-4.15.0-156-generic
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
root@vultr:~# sudo add-apt-repository ppa:vbernat/haproxy-2.4 -y
Hit:1 https://apprepo.vultr.com/debian buster InRelease
Get:2 http://ppa.launchpad.net/vbernat/haproxy-2.4/ubuntu bionic InRelease [20.8 kB]
Hit:3 http://de.clouds.archive.ubuntu.com/ubuntu bionic InRelease
Hit:4 http://security.ubuntu.com/ubuntu bionic-security InRelease
Get:5 http://ppa.launchpad.net/vbernat/haproxy-2.4/ubuntu bionic/main i386 Packages [976 B]
Get:6 http://ppa.launchpad.net/vbernat/haproxy-2.4/ubuntu bionic/main amd64 Packages [972 B]
Get:7 http://ppa.launchpad.net/vbernat/haproxy-2.4/ubuntu bionic/main Translation-en [704 B]
Hit:8 http://de.clouds.archive.ubuntu.com/ubuntu bionic-updates InRelease
Hit:9 http://de.clouds.archive.ubuntu.com/ubuntu bionic-backports InRelease
Fetched 23.4 kB in 1s (40.1 kB/s)
Reading package lists... Done
```
```
3. sudo apt install haproxy=2.4.\*
```
Result =>
```
Reading package lists... Done
Building dependency tree
Reading state information... Done
Selected version '2.4.17-1ppa1~bionic' (HAProxy 2.4:18.04/bionic [amd64]) for 'haproxy'
The following packages were automatically installed and are no longer required:
  linux-image-4.15.0-156-generic linux-modules-4.15.0-156-generic linux-modules-extra-4.15.0-156-generic
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libpcre2-8-0
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy libpcre2-8-0
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,763 kB of archives.
After this operation, 4,209 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ppa.launchpad.net/vbernat/haproxy-2.4/ubuntu bionic/main amd64 haproxy amd64 2.4.17-1ppa1~bionic [1,584 kB]
Get:2 http://de.clouds.archive.ubuntu.com/ubuntu bionic/universe amd64 libpcre2-8-0 amd64 10.31-2 [179 kB]
Fetched 1,763 kB in 1s (3,242 kB/s)
Selecting previously unselected package libpcre2-8-0:amd64.
(Reading database ... 146169 files and directories currently installed.)
Preparing to unpack .../libpcre2-8-0_10.31-2_amd64.deb ...
Unpacking libpcre2-8-0:amd64 (10.31-2) ...
Selecting previously unselected package haproxy.
Preparing to unpack .../haproxy_2.4.17-1ppa1~bionic_amd64.deb ...
Unpacking haproxy (2.4.17-1ppa1~bionic) ...
Setting up libpcre2-8-0:amd64 (10.31-2) ...
Setting up haproxy (2.4.17-1ppa1~bionic) ...
Installing new version of config file /etc/haproxy/errors/500.http ...
Installing new version of config file /etc/haproxy/haproxy.cfg ...
Installing new version of config file /etc/logrotate.d/haproxy ...
Installing new version of config file /etc/rsyslog.d/49-haproxy.conf ...
Processing triggers for libc-bin (2.27-3ubuntu1.6) ...
Processing triggers for systemd (237-3ubuntu10.53) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for rsyslog (8.32.0-1ubuntu4.2) ...
Processing triggers for ureadahead (0.100.0-21) ...
```
> You should now have the latest version of 2.4.x installed

Ensure that your haproxy installation is working and is running
```
4. service haproxy status
```
Result =>
```
* haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2022-07-14 19:03:36 UTC; 3min 43s ago
     Docs: man:haproxy(1)
           file:/usr/share/doc/haproxy/configuration.txt.gz
 Main PID: 721 (haproxy)
    Tasks: 3 (limit: 2314)
   CGroup: /system.slice/haproxy.service
           |-721 /usr/sbin/haproxy -sf 725 -x /run/haproxy/admin.sock -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
           `-775 /usr/sbin/haproxy -sf 725 -x /run/haproxy/admin.sock -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock
```
There are four essential sections to an HAProxy configuration file. They are **global**, **defaults**, **frontend**, and **backend**. These four sections define how the server as a whole performs, what your default settings are, and how client requests are received and routed to your backend servers.

The structure is as follows
```
global
    # global settings here

defaults
    # defaults here

frontend
    # a frontend that accepts requests from clients

backend
    # servers that fulfill the requests
```
The following section will explain a typical configuration for **Wax node** use

> **_NOTE:_** That the only difference here from the default configuration for the global section provided by the template is the enablement of CORS. This will be explained in more detail later

Global and Defaults: 
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        lua-load /etc/haproxy/cors.lua
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        #ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
```
> **_NOTE:_** This is a basic example you need to allow both http/s traffic to be accepted to be offloaded to the backend/s

Frontend:
```
frontend eoshttps
        bind *:443 ssl crt /etc/haproxy/certs alpn h2,http/1.1
        http-request set-header X-Forwarded-Proto https
        acl <identifier_for_rule> hdr(host) -i <hostheader_to_offload>
        http-request lua.cors
        http-response lua.cors "GET,PUT,POST" "*"
        use_backend <backend_system> if <identifier_for_rule>

frontend eoshttp
        bind *:80
        mode http
        acl <identifier_for_rule> hdr(host) -i <hostheader_to_offload>
        http-request lua.cors
        http-response lua.cors "GET,PUT,POST" "*"
        use_backend <backend_system> if <identifier_for_rule>
```
Explanation for the <span style="color:green">**values**</span> above:

- <span style="color:green">**identifier_for_rule**</span> can be any rule name you provide
- <span style="color:green">**backend_system**</span> can be any name of the backend system you will configure next so that the front-end requests know where to offload data
- <span style="color:green">**hostheader_to_offload**</span> the hostheader received by the call made to the proxy
- For tls communication, a certificate needs to be loaded so that the traffic between the (public client) and the proxy can be encrypted. In the configuration above, the certificate needs to be placed in <span style="color:green">**/etc/haproxy/certs**</span>

> **_NOTE:_** In this example I am offloading to 2 hyperion instances, both with their API's exposed on port 7000

Backend:
```
backend <backend_system>
        server wax 172.168.40.100:7000 maxconn 3000
        server wax 172.168.40.101:7000 maxconn 3000
```

### Rate limiting
> Rate limiting can reduce your traffic and potentially improve throughput by reducing the number of records sent to a service over a given period of time. <br>
> This can also be used to mitigate abuse where your frontend and backends will be overloaded by abusers, affecting the availability of your service <br> 

#### Setting the Maximum Connections
Use the maxconn parameter on a server line to cap the number of concurrent connections that will be sent. **Look at the example above which limits it too 3000 connections**

If all 30 connections are being used on all three servers, or in other words 90 connections are active, then new connections will have to wait in line for a slot to free up. This means that the servers themselves won’t become overloaded.