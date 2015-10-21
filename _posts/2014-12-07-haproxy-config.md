---
layout: post
title: "CentOS 下haproxy的安装与配置"
date: 2014-12-07
categories: linux
tags: haproxy centos 运维
---

### 1. 安装openssl
    tar -zxf openssl-1.0.1p.tar.gz
    ./config && make && make install

### 2. 安装haproxy
    make TARGET=linux26 USE_OPENSSL=1
    make install
    mkdir /etc/haproxy/
    mkdir -p /usr/local/haproxy/
    cp examples/haproxy.init /etc/init.d/haproxy
    cp examples/haproxy.cfg /etc/haproxy/


### 3. 配置haproxy
    global
           maxconn 20480
           log 127.0.0.1 local3
           chroot /usr/local/haproxy
           uid 0
           gid 0
           daemon
           nbproc 1
           pidfile /usr/local/haproxy/haproxy.pid
           ulimit-n 65535

    defaults
           log global
           mode http
           maxconn 20480
           option httplog
           option httpclose
           option dontlognull
           option forwardfor
           option redispatch
           option abortonclose
           stats refresh 30
           retries 3
           balance roundrobin

           timeout connect 5000
           timeout client 50000
           timeout server 50000
           timeout check 2000

    frontend http_in
           bind *:80
           #bind *:443 ssl crt /etc/haproxy/myallways.pem
           #redirect scheme https if !{ ssl_fc }
           mode http
           log global
           option httplog
           option httpclose
           option forwardfor

           acl host_open hdr(host)  -i open.myallways.cn

           default_backend server_default

    backend server_open
           mode http
           balance roundrobin
           cookie SERVERID
           option httpchk GET /index.html
           server open1 10.108.6.42:80 cookie open1 check inter 1500 rise 3 fall 3 weight 1
           server open2 10.108.6.46:80 cookie open2 check inter 1500 rise 3 fall 3 weight 1

    backend server_default
           mode http
           balance roundrobin
           cookie SERVERID
           option httpchk GET /index.html
           server default1 10.108.6.41:80 cookie default1 check inter 1500 rise 3 fall 3 weight 1
           server default2 10.108.6.45:80 cookie default2 check inter 1500 rise 3 fall 3 weight 1

### 4. 启动haproxy
    /usr/local/sbin/haproxy -f /etc/haproxy/haproxy.cfg -st `cat /usr/local/haproxy/haproxy.pid`

### 5. 同步haproxy到其他机器
    /usr/bin/rsync -avzP --progress --delete /usr/ root@10.108.6.44:/usr/