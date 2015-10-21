---
layout: post
title: "nginx 配置文件(Tengine)"
date: 2014-12-07
categories: linux
tags: nginx centos 运维
---

服务器现有配置备忘:

    user nginx;

    worker_processes  8;

    worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

    error_log  /usr/local/nginx/logs/error.log;
    pid        /usr/local/nginx/logs/nginx.pid;
    worker_rlimit_nofile 65535;


    events {
        use   epoll;
        worker_connections  65535;
    }

    http {

          include       mime.types;
          default_type  application/octet-stream;

          log_format access '$remote_addr - $remote_user [$time_local] "$request"' '$status $body_bytes_sent "$http_referer"' '"$http_user_agent" "$http_x_forwarded_for"';
          access_log    /usr/local/nginx/logs/access.log access;
          sendfile        on;

          tcp_nopush      on;
          tcp_nodelay     on;

          server_tag   off;
          server_info off;
          server_tokens   off;

          keepalive_timeout  10;
          client_header_timeout 10;
          client_body_timeout 10;
          reset_timedout_connection on;
          send_timeout 10;

          proxy_connect_timeout 30;
          proxy_send_timeout 20;
          proxy_read_timeout 60;
          proxy_buffering on;
          proxy_buffer_size 256k;
          proxy_buffers 4 256k;
          proxy_busy_buffers_size 512k;
          proxy_max_temp_file_size 2048m;
          proxy_temp_file_write_size 512k;
          proxy_headers_hash_max_size 51200;
          proxy_headers_hash_bucket_size 6400;

          gzip on;
          gzip_min_length 1k;
          gzip_buffers 4 16k;
          gzip_http_version 1.1;
          gzip_comp_level 6;
          gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript image/png image/jpeg;
          gzip_vary on;


          open_file_cache max=100000 inactive=20s;
          open_file_cache_valid 30s;
          open_file_cache_min_uses 2;
          open_file_cache_errors on;

          client_body_buffer_size       128k;

          client_header_buffer_size      32k;

          large_client_header_buffers  4 32k;

          server_names_hash_bucket_size  128;
          client_max_body_size 10m;



          upstream ilptest_dynamic {
           server 127.0.0.1:8090;
          }

          server {
               listen       80;
               server_name ilptest2.myallways.cn;
           access_log  logs/ilptest2.myallways.cn.access.log access;
           location ~ (mobile|resources).*
               {
                   expires 3d;
               add_header X-Cache '$upstream_cache_status from $server_addr';
               root /data/weball/anjiilp/WebRoot/;
               rewrite ^/anjiilp/(mobile|resources)/(.*)$ /$1/$2;
               }
               location / {
                   proxy_redirect off;
               proxy_set_header Host $host;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Forwarded-For $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_pass http://ilptest_dynamic;
               }
          }
    }
