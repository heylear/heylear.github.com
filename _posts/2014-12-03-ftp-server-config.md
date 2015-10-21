---
layout: post
title: "CentOS 下ftp服务器的搭建"
date: 2014-12-03
categories: linux
tags: vsftp centos ftp
---

vsftp 一般centos自带，安装方法不细说了。

### 1. 建立虚拟用户
    vim /tmp/ftp_login.txt

    anjiilp
    password
    anjiilpd
    password
    anjiilpapi
    password
    readonly
    readonly


    db_load -T -t hash -f /tmp/ftp_login.txt /etc/vsftpd/vsftpd_login.db

### 2. 配置/etc/pam.d/vsftpd.vu

    auth required pam_userdb.so db=/etc/vsftpd/vsftpd_login
    account required pam_userdb.so db=/etc/vsftpd/vsftpd_login


### 3. 配置/etc/vsftpd/vsftpd.conf

    anonymous_enable=NO
    local_enable=YES
    \#write_enable=NO
    dirmessage_enable=YES
    xferlog_enable=YES
    xferlog_file=/var/log/vsftpd.log
    connect_from_port_20=YES
    xferlog_std_format=YES
    listen=YES
    userlist_enable=YES
    chroot_local_user=YES
    tcp_wrappers=YES
    guest_enable=YES
    guest_username=ftpuser
    pam_service_name=vsftpd.vu
    user_config_dir=/etc/vsftpd/vsftpd_user_conf
    virtual_use_local_privs=YES
    \#pasv_min_port=50000
    \#pasv_max_port=50010
    \#pasv_enable=yes
    \#max_clients=200
    \#max_per_ip=4
    \#idle_session_timeout=600
    \#ftpd_banner=Welcome to CentOS FTP Service.


### 4. 建立本地虚拟用户

    useradd -d /data/ftpserver -s /sbin/nologin ftpuser
    chown ftpuser /data/ftpserver && chmod 755 /data/ftpserver


### 5. 配置ftp用户

    mkdir /etc/vsftpd/vsftpd_user_conf

    vim /etc/vsftpd/vsftpd_user_conf/anjiilp

    write_enable=YES
    anon_world_readable_only=NO
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    anon_other_write_enable=YES
    local_umask=022
    download_enable=NO
    local_root=/data/ftpserver/anjiilp

    vim /etc/vsftpd/vsftpd_user_conf/anjiilpd

    write_enable=YES
    anonymous_enable=NO
    anon_world_readable_only=NO
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    anon_other_write_enable=YES
    local_umask=022
    download_enable=YES
    local_root=/data/ftpserver/anjiilpd

    vim /etc/vsftpd/vsftpd_user_conf/anjiilpapi

    write_enable=YES
    anonymous_enable=NO
    anon_world_readable_only=NO
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    anon_other_write_enable=YES
    local_umask=022
    download_enable=YES
    local_root=/data/ftpserver/anjiilpapi

    vim /etc/vsftpd/vsftpd_user_conf/readony

    write_enable=NO
    anon_world_readable_only=NO
    anon_upload_enable=NO
    anon_mkdir_write_enable=NO
    anon_other_write_enable=NO
    local_umask=022
    download_enable=YES
    local_root=/data/ftpserver
