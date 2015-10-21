---
layout: post
title: "pexpect 无交互远程执行命令"
date: 2014-12-23
categories: python
tags: python pexpect 运维
---

服务器现有配置备忘:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    """
    This runs a command on a remote host using SSH. At the prompts enter hostname,
    user, password and the command.
    """

    import pexpect
    import getpass, os, sys

    #user: ssh 主机的用户名
    #host: ssh 主机的域名
    #port: ssh 主机的端口, 默认为22
    #password：ssh 主机的密码
    #command：即将在远端 ssh 主机上运行的命令
    def ssh_command (host, port, password, src, dest):

        child = pexpect.spawn('scp -l 500000 -P %s %s %s:%s'%(port, src ,host, dest))

        i = child.expect([ pexpect.TIMEOUT, 'yes/no', 'password:'])

        child.logfile = sys.stdout

        if ( i == 1):
            child.sendline('yes\r')

            i = child.expect([ pexpect.TIMEOUT, 'password:'])

        if ( i == 0):
            print '无法连接到远程主机'
            return None

        child.sendline(password)

        return child

    if __name__ == '__main__':

        if( len(sys.argv) != 5 ):
            print "错误的参数类型"
            os._exit(1)

        sshuri = sys.argv[1]

        if ( ':' in sshuri ):
            uriarr = sshuri.split(':')
            host = uriarr[0]
            port = uriarr[1]
        else:
            host = sshuri
            port = '22'

        password = sys.argv[2]

        src = sys.argv[3]

        dest = sys.argv[4]

        try:

            child = ssh_command(host, port, password, src, dest)

            child.expect([pexpect.EOF, pexpect.TIMEOUT],timeout=600)

            child.close()

            if ( child.exitstatus != 0 ):
                print child.before
                os._exit(child.exitstatus)

        except Exception, e:
            print "excep"
            print child.before
            print str(e)
            os._exit(1)
