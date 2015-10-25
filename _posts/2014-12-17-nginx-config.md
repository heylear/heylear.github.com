---
layout: post
title: "nginx 配置文件(Tengine)"
date: 2014-12-17
categories: linux
tags: nginx centos 运维
---

服务器现有配置备忘:

  user nginx;    
  #启动进程,通常设置成和cpu的数量相等
  worker_processes  8;

  #Nginx默认没有开启利用多核CPU,通过增加worker_cpu_affinity配置参数来充分利用多核CPU 以下是8核的配置参数
  worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
  #全局错误日志及PID文件
  error_log  /usr/local/nginx/logs/error.log;
  pid        /usr/local/nginx/logs/nginx.pid;
  worker_rlimit_nofile 65535;

  #工作模式及连接数上限
  events {
      use   epoll;                #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
      worker_connections  65535;  #单个后台worker process进程的最大并发链接数
                                  #事件模块指令,定义nginx每个进程最大连接数,默认1024。最大客户连接数由worker_processes和worker_connections决定
                                  #即 max_client=worker_processes*worker_connections,在作为反向代理时：max_client=worker_processes*worker_connections / 4
  }

  #设定http服务器,利用它的反向代理功能提供负载均衡支持
  http {
        #设定mime类型,类型由mime.type文件定义
        include       mime.types;
        default_type  application/octet-stream;
        #设定日志格式
        log_format access '$remote_addr - $remote_user [$time_local] "$request"' '$status $body_bytes_sent "$http_referer"' '"$http_user_agent" "$http_x_forwarded_for"';
        access_log    /usr/local/nginx/logs/access.log access;
        sendfile        on;        #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件,对于普通应用 必须设为 on
                                   #sendfile()是立即将数据从磁盘读到OS缓存。因为这种拷贝是在内核完成的，sendfile()要比组合read()和write()以及打开关闭丢弃缓冲更加有效
                                   #如果用来进行下载等应用磁盘IO重负载应用,可设置为 off,以平衡磁盘与网络I/O处理速度,降低系统的uptime.
        tcp_nopush      on;        #告诉nginx在一个数据包里发送所有头文件，而不一个接一个的发送
        tcp_nodelay     on;        #告诉nginx不要缓存数据，而是一段一段的发送--当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值
        server_tokens   off;       #错误时关闭显示nginx具体版本号这样安全
    
        ##连接客户端超时时间各种参数设置##
        keepalive_timeout  10;           #客户端连接时时间,超时之后服务器端自动关闭该连接 如果nginx守护进程在这个等待的时间里，一直没有收到浏览发过来http请求，则关闭这个http连接
        client_header_timeout 10;        #客户端请求头的超时时间
        client_body_timeout 10;          #客户端请求主体超时时间
        reset_timedout_connection on;    #告诉nginx关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间
        send_timeout 10;                 #客户端响应超时时间, 在两次客户端读取操作之间。如果在这段时间内，客户端没有读取任何数据，nginx就会关闭连接
        ################################
    
        ##代理设置 以下设置是nginx和后端服务器之间通讯的设置##
        proxy_connect_timeout 30;                      #nginx跟后端服务器连接超时时间(代理连接超时) 即发起握手等候响应的超时时间
        proxy_send_timeout 20;                         #后端服务器数据回传时间(代理发送超时) 指定后端server的数据回传时间,即在规定时间内后端服务器必须传完所有数据,否则nginx将断开连接
        proxy_read_timeout 60;                         #连接成功后,后端服务器响应时间(代理接收超时) #设置nginx从后端server取信息的时间,表示建立连接成功后,nginx等待后端服务器的响应时间
        proxy_buffering on;                            #该指令开启从后端被代理服务器的响应内容缓冲 此参数开启后proxy_buffers和proxy_busy_buffers_size参数才会起作用
        proxy_buffer_size 256k;                        #设置代理服务器（nginx）保存用户头信息的缓冲区大小  设置缓冲区大小,默认该缓冲区大小等于proxy_buffers当中的设置单个缓冲区的大小
        proxy_buffers 4 256k;                          #proxy_buffers缓冲区,网页平均在32k以下的话,这样设置 256k表示单个缓存区大小为265k
        proxy_busy_buffers_size 512k;                  #高负荷下缓冲大小（proxy_buffers*2） #设置系统很忙时可以使用的proxy_buffers大小,官方推荐的大小为proxy_buffers*2
        proxy_max_temp_file_size 2048m;                #临时文件的总大小，默认1024m
        proxy_temp_file_write_size 512k;               #设定缓存文件夹大小,大于这个值,将从upstream 服务器传递请求，而不缓冲到磁盘上
        proxy_headers_hash_max_size 51200;
        proxy_headers_hash_bucket_size 6400;
        #######################################################



        ###作为代理缓存服务器设置#######
        #proxy_temp_path  /var/tmp/nginx/proxy_temp;    ##定义缓存存储目录,之前必须要先手动创建此目录 此目录可以创建在系统缓存目录中
        #proxy_cache_path /var/tmp/nginx/proxy_cache levels=1:2 keys_zone=cache_one:512m inactive=10m max_size=64m;
        ###以上proxy_temp和proxy_cache需要在同一个分区中
        ###levels=1:2 表示缓存级别,表示缓存目录的第一级目录是1个字符，第二级目录是2个字符 keys_zone=cache_one:128m 缓存空间起名为cache_one 大小为512m
        ###max_size=64m 表示单个文件超过128m就不缓存了  inactive=10m 表示缓存的数据，10分钟内没有被访问过就删除 
        #########end####################



        #####对传输文件压缩###########
        gzip on;                 #开启gzip压缩
        gzip_min_length 1k;      #设置允许压缩的页面最小字节数,页面字节数从header头的Content-Length中获取。默认是0,表示不管多大都进行压缩,建议设置成大于1k的字节数,小于1k可能会越压越大。
        gzip_buffers 4 16k;      #表示申请4个为16k的内存作为存储gzip的压缩缓存
        gzip_http_version 1.1;   #指定http协议版本,默认1.1,默认即可
        gzip_comp_level 6;       #gzip压缩比,1为最小,处理最快；9为压缩比最大,处理最慢,传输速度最快,也最消耗CPU；
        gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript image/png image/jpeg;   #指定压缩类型,text/html这种类型即使不指定也会被压缩。
        gzip_vary on;            #让前端缓存server（如squid、varnish等）也缓存经gzip压缩的页面
        ##############################  

        open_file_cache max=100000 inactive=20s;    #打开缓存的同时也指定了缓存最大数目,以及缓存的时间。我们可以设置一个相对高的最大时间,这样我们可以在它们不活动超过20秒后清除掉
        open_file_cache_valid 30s;                  #在open_file_cache中指定检测正确信息的间隔时间
        open_file_cache_min_uses 2;                 #定义了open_file_cache中指令参数不活动时间期间里最小的文件数
        open_file_cache_errors on;                  #指定了当搜索一个文件时是否缓存错误信息,也包括再次给配置中添加文件。我们也包括了服务器模块,这些是在不同文件中定义的。如果你的服务器模块不在这些位置,你就得修改这一行来指定正确的位置

       

        ######设定客户端请求缓冲以及请求大小限制#####################
        client_body_buffer_size       128k;         #这个指令可以指定连接请求使用的缓冲区大小。如果连接请求超过缓存区指定的值，那么这些请求或部分请求将尝试写入一个临时文件。

        client_header_buffer_size      32k;         #这个指令指定客户端请求的http头部缓冲区大小,绝大多数情况下一个头部请求的大小不会大于1k不过如果有
                                                    #来自于wap客户端的较大的cookie它可能会大于1k，Nginx将分配给它一个更大的缓冲区，这个值可以在large_client_header_buffers里面设置。

        large_client_header_buffers  4 32k;         #指定客户端请求头中较大的消息头缓存最大数量和大小 如果一个请求的URI大小超过这个值
                                                    #服务器将返回一个"Request URI too large" (414)，同样，如果一个请求的头部字段大于这个值，服务器将返回"Bad request" (400)。
                #缓冲区根据需求的不同是分开的。默认一个缓冲区大小为操作系统中分页文件大小，通常是4k或8k，
                #如果一个连接请求将状态转换为keep-alive，这个缓冲区将被释放。

        server_names_hash_bucket_size  128;
        client_max_body_size 10m;                   #这个指令指定允许客户端请求的最大的单个文件字节数，它出现在请求头部的Content-Length字段。
                                                    #如果请求大于指定的值，客户端将收到一个"Request Entity Too Large" (413)错误。
        #############################################################


        #设定负载均衡的服务器列表
        #upstream ilptest_static {
        #          #以下是轮询模式
        #          #weigth参数表示权值,权值越高被分配到的几率越大
        #          #server 10.101.202.201:8088  weight=1;
        #          #server 10.101.202.202:8088  weight=1;
        #          ##以下是ip_hash模式
        #          #ip_hash;
        #          #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
        #          #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
        #          #sticky;
        #          server 10.104.0.100:9091;
        #          server 10.104.0.38:9091;
        #          server 10.104.0.39:9091;
        #}

        #设定负载均衡的服务器列表
        upstream ilp_dynamic {
                  #以下是轮询模式
                  #weigth参数表示权值,权值越高被分配到的几率越大
                  #server 10.101.202.201:8088  weight=1;
                  #server 10.101.202.202:8088  weight=1;
                  ##以下是ip_hash模式
                  #ip_hash;
                  #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
                  #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
                  #sticky;
                  server 10.104.0.165:9081;
        }

        #设定负载均衡的服务器列表
        upstream ilpd_dynamic {
                  #以下是轮询模式
                  #weigth参数表示权值,权值越高被分配到的几率越大
                  #server 10.101.202.201:8088  weight=1;
                  #server 10.101.202.202:8088  weight=1;
                  ##以下是ip_hash模式
                  #ip_hash;
                  #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
                  #server  10.101.202.201:8088  max_fails=3 fail_timeout=10s;
                  #sticky;
                  server 10.104.0.165:9080;
        }
      
        server {
          #侦听80端口
          listen       80;
          #定义使用haha.mysvw.com访问
          server_name ilptest2.myallways.cn;

          #设定本虚拟主机的访问日志
    access_log  logs/ilptest2.myallways.cn.access.log access;             #使用access格式来记录日志,access格式在之前LogFormat中已经定义过了
    #location ~ /purge(/.*)   #用于清除缓存 清除方法要清除http://IP/images/aaa.jpg 方法如下输入http://IP/purge/images/aaa.jpg即可
          #{
          #    allow                127.0.0.1;
          #    allow                10.101.202.0/24;   #设置只允许指定的IP或IP段才可以清除URL缓存。
    #    allow                10.169.100./24;
          #    deny                 all;
    #    proxy_cache_purge    cache_one   $host$1$is_args$args;
          #}

    ##访问图片的请求##
          #location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css|html)$
    #location ~ (mobile|resources).*
          #{
    #    #proxy_cache cache_one;                                           #开启使用名为cache_one的缓存区
          #    #proxy_cache_valid  200 304 302 1d;                               ####说明对于http返回码状态为200和304以及302的缓存文件的缓存时间是1天
    #                                                                     #####1天之后再访问该缓存文件时,文件会过期从而去源服务器重新取数据
          #    #proxy_cache_valid  any 10h;                                      ##表示除了以上200/304/302返回码之外 缓存时间为10小时
          #    #proxy_cache_use_stale updating;                                  #updating参数允许nginx在正在更新缓存的情况下使用过期的缓存作为响应。这样做可以使更新缓存数据时，访问源服务器的次数最少
          #    #proxy_cache_key $host$uri$is_args$args;                          #由主机名、唯一资源定位符、参数判断符和请求参数共同生成哈希缓存的key
    #
          #    expires 3d;                                                      ##在屏蔽掉服务器端的Cache-Control以及Expires的http回应头的前提下 此expires的设置 就是等于设置了max-age返回给浏览器
    #                                                                     ##告诉浏览器此资源在客户端的浏览器缓存中需要缓存3天,而在这3天内用户通过此浏览器访问此资源时其实其http包都没有
    #                                            ##发出去,而是直接请求本地浏览器的缓存了(此前提是用户不能在浏览器上按F5或CTRL+F5)
    #    add_header X-Cache '$upstream_cache_status from $server_addr';   ##在响应头中增加一个自定义字段叫X-Cache 内容如HIT from IP/MISS from IP/EXPIRED from IP
    #
    #    ######以下这段设置目的是屏蔽掉由后端源服务器返回给nginx时的相关指定响应头字段#################
    #    proxy_ignore_headers  "Cache-Control";
          #    proxy_hide_header     "Cache-Control";
    #
          #    proxy_ignore_headers  "Expires";
          #    proxy_hide_header     "Expires";
    #
          #    proxy_hide_header     "Set-Cookie";
          #    proxy_ignore_headers  "Set-Cookie";
    #    ##############################################################################################
    #
    #    ####以下proxy_set_header命令目的是通过从客户端请求协议头传给nginx后再通过nginx加工修改相关协议头参数信息并以请求头的身份转发给后端服务器使用##########
    #    proxy_set_header Host $host;                                     #将用户在地址栏输入的host名字(如bbs.sina.com)通过nginx传递给后端服务器使用(如果后端服务器有多个虚拟主机名字的话,
    #                                                                     #例如后端服务器虚拟主机既有blog.sina.com又配置了bbs.sina.com 但是后端服务器IP是一样的就是虚机主机名字不同)
    #                                                                     #当后端单台web服务器上也配置有多个虚拟主机时,需要使用该header来区分反向代理哪个主机名
          #    proxy_set_header X-Real-IP $remote_addr;                         #获取用户真实IP地址
          #    proxy_set_header X-Forwarded-For $remote_addr;                   #将$remote_addr变量即用户真实IP地址 替换X-Forwarded-For参数的参数值 目的是清空了客户端可能伪造传入的X-Forwarded-For
          #                                                                     #保证了获取的ip为真实ip，
          #    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;     #以上一条指令和这条指令 一起作用 目的是防止X-Forwarded-For参数欺骗
    #                                                                     #$proxy_add_x_forwarded_for变量包含客户端请求头中的"X-Forwarded-For",与$remote_addr两部分,他们之间用逗号分开.
    #    ##############################################################################################
          #    proxy_pass http://ilptest_static;
          #}
    
    #默认请求
          location /anjiilpd/ {
              proxy_redirect off;                                              #如果需要修改从被代理server传来的应答头中的"Location"和"Refresh"字段,可以用这个指令设置

        ####以下proxy_set_header命令目的是通过从客户端请求协议头传给nginx后再通过nginx加工修改相关协议头参数信息并以请求头的身份转发给后端服务器使用##########
        proxy_set_header Host $host;                                     #将用户在地址栏输入的host名字(如bbs.sina.com)通过nginx传递给后端服务器使用(如果后端服务器有多个虚拟主机名字的话,
                                                                         #例如后端服务器虚拟主机既有blog.sina.com又配置了bbs.sina.com 但是后端服务器IP是一样的就是虚机主机名字不同)
                                                                         #当后端单台web服务器上也配置有多个虚拟主机时,需要使用该header来区分反向代理哪个主机名
              proxy_set_header X-Real-IP $remote_addr;                         #获取用户真实IP地址
              proxy_set_header X-Forwarded-For $remote_addr;                   #将$remote_addr变量即用户真实IP地址 替换X-Forwarded-For参数的参数值 目的是清空了客户端可能伪造传入的X-Forwarded-For
                                                                               #保证了获取的ip为真实ip，
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;     #以上一条指令和这条指令 一起作用 目的是防止X-Forwarded-For参数欺骗
                                                                         #$proxy_add_x_forwarded_for变量包含客户端请求头中的"X-Forwarded-For",与$remote_addr两部分,他们之间用逗号分开.
              ######################################################################################################################################################
        proxy_pass http://ilpd_dynamic;
          }

    #默认请求
          location / {
              proxy_redirect off;                                              #如果需要修改从被代理server传来的应答头中的"Location"和"Refresh"字段,可以用这个指令设置

        ####以下proxy_set_header命令目的是通过从客户端请求协议头传给nginx后再通过nginx加工修改相关协议头参数信息并以请求头的身份转发给后端服务器使用##########
        proxy_set_header Host $host;                                     #将用户在地址栏输入的host名字(如bbs.sina.com)通过nginx传递给后端服务器使用(如果后端服务器有多个虚拟主机名字的话,
                                                                         #例如后端服务器虚拟主机既有blog.sina.com又配置了bbs.sina.com 但是后端服务器IP是一样的就是虚机主机名字不同)
                                                                         #当后端单台web服务器上也配置有多个虚拟主机时,需要使用该header来区分反向代理哪个主机名
              proxy_set_header X-Real-IP $remote_addr;                         #获取用户真实IP地址
              proxy_set_header X-Forwarded-For $remote_addr;                   #将$remote_addr变量即用户真实IP地址 替换X-Forwarded-For参数的参数值 目的是清空了客户端可能伪造传入的X-Forwarded-For
                                                                               #保证了获取的ip为真实ip，
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;     #以上一条指令和这条指令 一起作用 目的是防止X-Forwarded-For参数欺骗
                                                                         #$proxy_add_x_forwarded_for变量包含客户端请求头中的"X-Forwarded-For",与$remote_addr两部分,他们之间用逗号分开.
              ######################################################################################################################################################
        proxy_pass http://ilp_dynamic;
          }

          #定义错误提示页面
          #error_page 500 502 503 504 /50x.html;                                #只要返回码为500 502 503 504 都将url转到/50x.html页面
          #location = /50x.html  {                                              #
          #    root   /var/www;                                                 #和上面一行一起作用 用途是定义/50x.html文件在服务器的/var/www路径下面
          #}

          #设定查看Nginx状态的地址
          #location /NginxStatus {
          #    stub_status             on;
          #    access_log              on;
          #    auth_basic              "NginxStatus";
          #    auth_basic_user_file    htpasswd;
          #}

          #禁止访问 .htxxx 文件
          #location ~ /\.ht {
          #    deny all;
          #}
       
        }
  }

