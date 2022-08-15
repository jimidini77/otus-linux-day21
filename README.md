# otus-linux-day21
Zabbix

# **Prerequisite**
- Host OS: Windows 10.0.19043
- Guest OS: CentOS 8.3.2011
- VirtualBox: 6.1.36
- Vagrant: 2.2.19
- Zabbix: 6.2.1
- MySQL: 8.0.26

# **Содержание ДЗ**

* Настройка мониторинга. На сервере установить Zabbix, настроить дашборд с 4-мя графиками:
  - процессор;
  - память;
  - диск;
  - сеть;

# **Выполнение**

## Установка Zabbix
Настройка репозиториев:
```sh
[root@zabbix ~]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@zabbix ~]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```
Установка, настройка и тестирование MySQL:
```sh
[root@zabbix ~]# dnf install -y mysql-server
[root@zabbix ~]# systemctl start mysqld.service
[root@zabbix ~]# systemctl enable mysqld
[root@zabbix ~]# mysqladmin -u root -p version
Enter password:
mysqladmin  Ver 8.0.26 for Linux on x86_64 (Source distribution)
Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version          8.0.26
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 2 min 12 sec

Threads: 2  Questions: 2  Slow queries: 0  Opens: 120  Flush tables: 3  Open tables: 36  Queries per second avg: 0.015
```
Настройка репозитория Zabbix:
```sh
[root@zabbix ~]# rpm -Uvh https://repo.zabbix.com/zabbix/6.2/rhel/8/x86_64/zabbix-release-6.2-1.el8.noarch.rpm
[root@zabbix ~]# dnf clean all
```
Переключение версии DNF модуля для PHP:
```sh
[root@zabbix ~]# dnf module enable -y php:7.4
Last metadata expiration check: 0:07:36 ago on Mon 15 Aug 2022 09:52:26 AM UTC.
Dependencies resolved.
========================================================================================================================
 Package                     Architecture               Version                       Repository                   Size
========================================================================================================================
Enabling module streams:
 httpd                                                  2.4
 nginx                                                  1.14
 php                                                    7.4

Transaction Summary
========================================================================================================================

Complete!
```

Установка Zabbix сервера, веб-интерфейса и агента:
```sh
[root@zabbix ~]# dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-nginx-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

Создание базы данных:
```sql
[root@zabbix ~]# mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> SET GLOBAL log_bin_trust_function_creators = 1;
mysql> quit;
```

Импорт начальной схемы и данных:
```sh
[root@zabbix ~]# zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```
```sql
[root@zabbix ~]# mysql -uroot -p
password
mysql> SET GLOBAL log_bin_trust_function_creators = 0;
mysql> quit;
```

Настройка БД для Zabbix. В `/etc/zabbix/zabbix_server.conf`:
```
DBPassword=password
```
Настройка веб-интерфейса. В `/etc/nginx/conf.d/zabbix.conf`:
```
listen 8080;
```

Запуск сервисов Zabbix сервера и агента:
```sh
[root@zabbix ~]# systemctl restart zabbix-server zabbix-agent nginx php-fpm
[root@zabbix ~]# systemctl enable zabbix-server zabbix-agent nginx php-fpm
```
Порты прослушиваются сервисами:
```sh
[root@zabbix ~]# ss -tnlp
State          Recv-Q          Send-Q                   Local Address:Port                    Peer Address:Port
LISTEN         0               128                            0.0.0.0:111                          0.0.0.0:*             users:(("rpcbind",pid=582,fd=4),("systemd",pid=1,fd=37))
LISTEN         0               128                            0.0.0.0:80                           0.0.0.0:*             users:(("nginx",pid=25498,fd=9),("nginx",pid=25497,fd=9))
LISTEN         0               128                            0.0.0.0:8080                         0.0.0.0:*             users:(("nginx",pid=25498,fd=8),("nginx",pid=25497,fd=8))
LISTEN         0               128                            0.0.0.0:22                           0.0.0.0:*             users:(("sshd",pid=729,fd=3))
LISTEN         0               128                            0.0.0.0:10050                        0.0.0.0:*             users:(("zabbix_agentd",pid=25492,fd=4),("zabbix_agentd",pid=25491,fd=4),("zabbix_agentd",pid=25490,fd=4),("zabbix_agentd",pid=25489,fd=4),("zabbix_agentd",pid=25488,fd=4),("zabbix_agentd",pid=25486,fd=4))
LISTEN         0               128                            0.0.0.0:10051                        0.0.0.0:*             users:(("zabbix_server",pid=25568,fd=11),("zabbix_server",pid=25567,fd=11),("zabbix_server",pid=25566,fd=11),("zabbix_server",pid=25565,fd=11),("zabbix_server",pid=25564,fd=11),("zabbix_server",pid=25563,fd=11),("zabbix_server",pid=25562,fd=11),("zabbix_server",pid=25561,fd=11),("zabbix_server",pid=25560,fd=11),("zabbix_server",pid=25559,fd=11),("zabbix_server",pid=25558,fd=11),("zabbix_server",pid=25557,fd=11),("zabbix_server",pid=25545,fd=11),("zabbix_server",pid=25544,fd=11),("zabbix_server",pid=25543,fd=11),("zabbix_server",pid=25542,fd=11),("zabbix_server",pid=25541,fd=11),("zabbix_server",pid=25540,fd=11),("zabbix_server",pid=25539,fd=11),("zabbix_server",pid=25538,fd=11),("zabbix_server",pid=25536,fd=11),("zabbix_server",pid=25535,fd=11),("zabbix_server",pid=25534,fd=11),("zabbix_server",pid=25533,fd=11),("zabbix_server",pid=25532,fd=11),("zabbix_server",pid=25531,fd=11),("zabbix_server",pid=25530,fd=11),("zabbix_server",pid=25529,fd=11),("zabbix_server",pid=25528,fd=11),("zabbix_server",pid=25527,fd=11),("zabbix_server",pid=25526,fd=11),("zabbix_server",pid=25525,fd=11),("zabbix_server",pid=25524,fd=11),("zabbix_server",pid=25523,fd=11),("zabbix_server",pid=25522,fd=11),("zabbix_server",pid=25521,fd=11),("zabbix_server",pid=25520,fd=11),("zabbix_server",pid=25519,fd=11),("zabbix_server",pid=25518,fd=11),("zabbix_server",pid=25517,fd=11),("zabbix_server",pid=25516,fd=11),("zabbix_server",pid=25515,fd=11),("zabbix_server",pid=25514,fd=11),("zabbix_server",pid=25513,fd=11),("zabbix_server",pid=25501,fd=11),("zabbix_server",pid=25500,fd=11),("zabbix_server",pid=25487,fd=11))
LISTEN         0               128                               [::]:111                             [::]:*             users:(("rpcbind",pid=582,fd=6),("systemd",pid=1,fd=39))
LISTEN         0               128                               [::]:80                              [::]:*             users:(("nginx",pid=25498,fd=10),("nginx",pid=25497,fd=10))
LISTEN         0               128                               [::]:22                              [::]:*             users:(("sshd",pid=729,fd=4))
LISTEN         0               128                               [::]:10050                           [::]:*             users:(("zabbix_agentd",pid=25492,fd=5),("zabbix_agentd",pid=25491,fd=5),("zabbix_agentd",pid=25490,fd=5),("zabbix_agentd",pid=25489,fd=5),("zabbix_agentd",pid=25488,fd=5),("zabbix_agentd",pid=25486,fd=5))
LISTEN         0               128                               [::]:10051                           [::]:*             users:(("zabbix_server",pid=25568,fd=12),("zabbix_server",pid=25567,fd=12),("zabbix_server",pid=25566,fd=12),("zabbix_server",pid=25565,fd=12),("zabbix_server",pid=25564,fd=12),("zabbix_server",pid=25563,fd=12),("zabbix_server",pid=25562,fd=12),("zabbix_server",pid=25561,fd=12),("zabbix_server",pid=25560,fd=12),("zabbix_server",pid=25559,fd=12),("zabbix_server",pid=25558,fd=12),("zabbix_server",pid=25557,fd=12),("zabbix_server",pid=25545,fd=12),("zabbix_server",pid=25544,fd=12),("zabbix_server",pid=25543,fd=12),("zabbix_server",pid=25542,fd=12),("zabbix_server",pid=25541,fd=12),("zabbix_server",pid=25540,fd=12),("zabbix_server",pid=25539,fd=12),("zabbix_server",pid=25538,fd=12),("zabbix_server",pid=25536,fd=12),("zabbix_server",pid=25535,fd=12),("zabbix_server",pid=25534,fd=12),("zabbix_server",pid=25533,fd=12),("zabbix_server",pid=25532,fd=12),("zabbix_server",pid=25531,fd=12),("zabbix_server",pid=25530,fd=12),("zabbix_server",pid=25529,fd=12),("zabbix_server",pid=25528,fd=12),("zabbix_server",pid=25527,fd=12),("zabbix_server",pid=25526,fd=12),("zabbix_server",pid=25525,fd=12),("zabbix_server",pid=25524,fd=12),("zabbix_server",pid=25523,fd=12),("zabbix_server",pid=25522,fd=12),("zabbix_server",pid=25521,fd=12),("zabbix_server",pid=25520,fd=12),("zabbix_server",pid=25519,fd=12),("zabbix_server",pid=25518,fd=12),("zabbix_server",pid=25517,fd=12),("zabbix_server",pid=25516,fd=12),("zabbix_server",pid=25515,fd=12),("zabbix_server",pid=25514,fd=12),("zabbix_server",pid=25513,fd=12),("zabbix_server",pid=25501,fd=12),("zabbix_server",pid=25500,fd=12),("zabbix_server",pid=25487,fd=12))
LISTEN         0               70                                   *:33060                              *:*             users:(("mysqld",pid=24654,fd=22))
LISTEN         0               128                                  *:3306                               *:*             users:(("mysqld",pid=24654,fd=25))
```
## Настройка дашборда

По настроенному адресу доступен интерфейс начального конфигурирования Zabbix:

(Config Page)[]

# **Результаты**

Выполнено развёртывание nginx на стенде с использованием Ansible и ролей.
Полученный в ходе работы `Vagrantfile` с настроенным Ansible provisioner помещен в публичный репозиторий.
При поднятии ВМ выполняется установка и настройка Nginx с использованием роли Ansible

- **GitHub** - https://github.com/jimidini77/otus-linux-day15
