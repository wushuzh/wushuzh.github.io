+++
date = "2017-08-17T21:14:17+08:00"
title = "Zabbix 监控"
showonlyimage = false
image = "/img/blog/dual-os-3-setup-arch/setup-Arch.png"
draft = true
weight = 65
+++

从 0 到 1 设定 Archlinux 桌面
<!--more-->

135.252.182.76

./kvm-install-vm -n zabbix-server -c 2 -m 2048

scp /etc/yum.repos.d/centos.repo centos@192.168.122.168:~

ssh -lcentos 192.168.122.168
sudo rm -f /etc/yum.repos.d/Cent*repo
sudo mv ~/centos.repo .

https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7

sudo yum install httpd
sudo systemctl start httpd.service
sudo systemctl enable httpd.service

sudo yum install mariadb-server mariadb
sudo systemctl start mariadb
sudo mysql_secure_installation
    root passwd mariadb
    enter to choose default for rest question

sudo systemctl enable mariadb.service


sudo yum install php php-mysql

$ yum search php-                                                                        [3/1920]
Loaded plugins: fastestmirror
Determining fastest mirrors
======================================================== N/S matched: php- ========================================================
php-bcmath.x86_64 : A module for PHP applications for using the bcmath library
php-cli.x86_64 : Command-line interface for PHP
php-common.x86_64 : Common files for PHP
php-dba.x86_64 : A database abstraction layer module for PHP applications
php-devel.x86_64 : Files needed for building PHP extensions
php-embedded.x86_64 : PHP library for embedding in applications
php-enchant.x86_64 : Enchant spelling extension for PHP applications
php-fpm.x86_64 : PHP FastCGI Process Manager
php-gd.x86_64 : A module for PHP applications for using the gd graphics library
php-intl.x86_64 : Internationalization extension for PHP applications
php-ldap.x86_64 : A module for PHP applications that use LDAP
php-mbstring.x86_64 : A module for PHP applications which need multi-byte string handling
php-mysql.x86_64 : A module for PHP applications that use MySQL databases
php-mysqlnd.x86_64 : A module for PHP applications that use MySQL databases
php-odbc.x86_64 : A module for PHP applications that use ODBC databases
php-pdo.x86_64 : A database access abstraction module for PHP applications
php-pear.noarch : PHP Extension and Application Repository framework
php-pecl-memcache.x86_64 : Extension to work with the Memcached caching daemon
php-pgsql.x86_64 : A PostgreSQL database module for PHP
php-process.x86_64 : Modules for PHP script using system process interfaces
php-pspell.x86_64 : A module for PHP applications for using pspell interfaces
php-recode.x86_64 : A module for PHP applications for using the recode library
php-snmp.x86_64 : A module for PHP applications that query SNMP-managed devices
php-soap.x86_64 : A module for PHP applications that use the SOAP protocol
php-xml.x86_64 : A module for PHP applications which use XML
php-xmlrpc.x86_64 : A module for PHP applications which use the XML-RPC protocol


vncserver

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-remote-access-for-the-gnome-desktop-on-centos-7
stop firewall enable vnc access
  https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7
  sudo firewall-cmd --permanent --zone=public --add-service=vnc-server



install zabbix according to do

but zabbix cannot start due to selinux
    https://support.zabbix.com/browse/ZBX-10542

    sudo yum install setroubleshoot

    https://www.digitalocean.com/community/tutorials/an-introduction-to-selinux-on-centos-7-part-1-basic-concepts
    https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Which_Log_File_is_Used.html

    https://www.server-world.info/en/note?os=CentOS_7&p=selinux&f=8

        [centos@zabbix-target log]$ sudo systemctl restart auditd.service
        Failed to restart auditd.service: Operation refused, unit auditd.service may be requested by dependency only.
        See system logs and 'systemctl status auditd.service' for details.

        service auditd restart

https://support.zabbix.com/browse/ZBX-10542

https://bugzilla.redhat.com/show_bug.cgi?id=1323518

https://bugzilla.redhat.com/show_bug.cgi?id=1393332

http://cn.linux.vbird.org/linux_server/0210network-secure_4.php

Job for zabbix-server.service failed because a configured resource limit was exceeded. See "systemctl status zabbix-server.service" and "journalctl -xe" for details.
