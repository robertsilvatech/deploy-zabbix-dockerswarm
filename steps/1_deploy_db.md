# Deploy DB

- [Deploy DB](#deploy-db)
  - [Timezone](#timezone)
    - [Check the timezone](#check-the-timezone)
    - [Set timezone](#set-timezone)
  - [Tuning S.O](#tuning-so)
    - [Check updates](#check-updates)
    - [Install utilities](#install-utilities)
    - [Desable SELINUX](#desable-selinux)
  - [Install MySQL 8](#install-mysql-8)
    - [Check version available in the Centos 8 repository](#check-version-available-in-the-centos-8-repository)
    - [Install the MySQL 8 package](#install-the-mysql-8-package)
    - [Enable automatic startup, start the service and validate the status](#enable-automatic-startup-start-the-service-and-validate-the-status)
    - [Set MySQL root user password](#set-mysql-root-user-password)
    - [Connect to mysql, create the Zabbix database and user](#connect-to-mysql-create-the-zabbix-database-and-user)
    - [Create zabbix user allowing connection through a remote server](#create-zabbix-user-allowing-connection-through-a-remote-server)
    - [Create firewall rule](#create-firewall-rule)

## Timezone

### Check the timezone

```bash
timedatectl status
```

Output:

```bash
               Local time: Wed 2020-06-03 05:03:25 -03
           Universal time: Wed 2020-06-03 08:03:25 UTC
                 RTC time: Wed 2020-06-03 08:03:24
                Time zone: America/New_York (-03, -0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

### Set timezone

```bash
timedatectl set-timezone America/Sao_Paulo
```
## Tuning S.O

### Check updates

```bash
dnf clean all
dnf check-update
```

### Install utilities

```bash
dnf install -y net-tools vim nano epel-release wget curl tcpdump telnet
```

### Desable SELINUX

`REBOOT REQUIRED`

```bash
cat /etc/selinux/config
vim /etc/selinux/config
SELINUX=disabled
```

`You can change the current state and restart later`

```bash
setenforce 0
sestatus
```

## Install MySQL 8

### Check version available in the Centos 8 repository

```bash
dnf info mysql-server
```

Output:

```bash
Extra Packages for Enterprise Linux Modular 8 - x86_64                                                                                                           228 kB/s | 118 kB     00:00
Extra Packages for Enterprise Linux 8 - x86_64                                                                                                                   7.3 MB/s | 6.9 MB     00:00
Last metadata expiration check: 0:00:01 ago on Mon 08 Jun 2020 11:47:25 PM -03.
Available Packages
Name         : mysql-server
Version      : 8.0.17
Release      : 3.module_el8.0.0+181+899d6349
Architecture : x86_64
Size         : 22 M
Source       : mysql-8.0.17-3.module_el8.0.0+181+899d6349.src.rpm
Repository   : AppStream
Summary      : The MySQL server and related files
URL          : http://www.mysql.com
License      : GPLv2 with exceptions and LGPLv2 and BSD
Description  : MySQL is a multi-user, multi-threaded SQL database server. MySQL is a
             : client/server implementation consisting of a server daemon (mysqld)
             : and many different client programs and libraries. This package contains
             : the MySQL server and some accompanying files and directories.
```

### Install the MySQL 8 package

```bash
dnf -y install mysql-server
```

### Enable automatic startup, start the service and validate the status

```bash
systemctl enable --now mysqld
systemctl status mysqld
```

### Set MySQL root user password

```bash
mysql_secure_installation

Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
Please set the password for root here.

New password: Z4bb1xSumm1t2020!

Re-enter new password: Z4bb1xSumm1t2020!

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
```

### Connect to mysql, create the Zabbix database and user


```bash
mysql -u root -p
create database zabbix character set utf8 collate utf8_bin;
create user 'zabbix'@'localhost' identified by 'Workshop!Zabbix5';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
```

`character set utf8 - support for multilingualism`
`collate utf8_bin - case sensitiveness of stored data`


### Create zabbix user allowing connection through a remote server

```bash
create user 'zabbix'@'10.0.0.62' identified with mysql_native_password by 'Workshop!Zabbix5';
grant all privileges on zabbix.* to 'zabbix'@'10.0.0.62';
UPDATE mysql.user SET Super_Priv='Y' WHERE user='zabbix' AND host='10.0.0.62';
create user 'zabbix'@'10.0.0.63' identified with mysql_native_password by 'Workshop!Zabbix5';
grant all privileges on zabbix.* to 'zabbix'@'10.0.0.63';
UPDATE mysql.user SET Super_Priv='Y' WHERE user='zabbix' AND host='10.0.0.63';
flush privileges;
create user 'zabbix'@'10.0.0.64' identified with mysql_native_password by 'Workshop!Zabbix5';
grant all privileges on zabbix.* to 'zabbix'@'10.0.0.64';
UPDATE mysql.user SET Super_Priv='Y' WHERE user='zabbix' AND host='10.0.0.64';
flush privileges;
quit
```

### Create firewall rule

```shell
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
```