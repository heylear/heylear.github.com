---
layout: post
title: "CentOs虚拟机安装及Mysql5.6源码安装"
date: 2014-11-01
comments: false
categories: linux
---

## 永久关闭防火墙
`chkconfig -–level 35 iptables off`

## 修改网上IP地址
<pre>
vi /etc/sysconfig/network-scripts/ifcfg-eth0
vi /etc/sysconfig/network
vi /etc/resolv.conf}
</pre>

## 后台启动与关闭虚拟机的方式
`vboxmanage controlvm centos poweroff
vboxmanage startvm centos --type headless`

注意: linux双网卡配置注意默认路由不能为Hostonly网卡，否则不能上外网

## linux安装mysql-5.6.22 
源码地址：[http://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.22.tar.gz](http://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.22.tar.gz)

## 安装编译源码所需工具
`yum install gcc-c++ cmake bison-devel  ncurses-devel`

## 配置安装选项
<pre>
cmake 
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql 
-DMYSQL_DATADIR=/usr/local/mysql/data 
-DSYSCONFDIR=/etc 
-DWITH_MYISAM_STORAGE_ENGINE=1 
-DWITH_INNOBASE_STORAGE_ENGINE=1 
-DWITH_MEMORY_STORAGE_ENGINE=1 
-DWITH_READLINE=1 
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock 
-DMYSQL_TCP_PORT=3306 
-DENABLED_LOCAL_INFILE=1 
-DWITH_PARTITION_STORAGE_ENGINE=1 
-DEXTRA_CHARSETS=all 
-DDEFAULT_CHARSET=utf8 
-DDEFAULT_COLLATION=utf8_general_ci
</pre>

## 安装命令
`make && make install`

注意：在CentOS 6.4版操作系统的最小安装完成后，在/etc目录下会存在一个my.cnf，需要将此文件更名为其他的名字，如：/etc/my.cnf.bak，否则，该文件会干扰源码安装的MySQL的正确配置，造成无法启动。

在使用"yum update"更新系统后，需要检查下/etc目录下是否会多出一个my.cnf，如果多出，将它重命名成别的。否则，MySQL将使用这个配置文件启动，可能造成无法正常启动等问题。