
# Centos7安装部署Mariadb数据库

```
版本 0.1
时间 2020.03.30 09:08
```


## 1. 安装
```SHELL
yum install mariadb-server -y
```

## 2. 配置Mariadb
1). 把MariaDB服务开启，并设置为开机启动  
```SHELL
systemctl start mariadb  # 开启服务
systemctl enable mariadb  # 设置为开机自启动服务
```

2). 首次安装需要进行数据库的配置，命令都和mysql的一样  
```SHELL
mysql_secure_installation
```

3). 配置时出现的各个选项  
```
Enter current password for root (enter for none):  # 输入数据库超级管理员root的密码(注意不是系统root的密码)，第一次进入还没有设置密码则直接回车

Set root password? [Y/n]  # 设置密码，y

New password:  # 新密码
Re-enter new password:  # 再次输入密码

Remove anonymous users? [Y/n]  # 移除匿名用户， y

Disallow root login remotely? [Y/n]  # 拒绝root远程登录，n，不管y/n，都会拒绝root远程登录

Remove test database and access to it? [Y/n]  # 删除test数据库，y：删除。n：不删除，数据库中会有一个test数据库，一般不需要

Reload privilege tables now? [Y/n]  # 重新加载权限表，y。或者重启服务也许
```

4). 测试是否能够登录成功，出现  MariaDB [(none)]> 就表示已经能够正常登录使用MariaDB数据库了  
```
[root@mini ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>

```

## 3.设置Mariadb字符集utf-8

1). /etc/my.cnf 文件, 在  [mysqld]  标签下添加  
```
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```

2). /etc/my.cnf.d/client.cnf 文件, 在  [client]  标签下添加  
```
default-character-set=utf8
```

3). /etc/my.cnf.d/mysql-clients.cnf  文件, 在  [mysql]  标签下添加  
```
default-character-set=utf8
```

4). 重启服务  
```
systemctl restart mariadb
```

5). 进入mariadb查看字符集  
```
MariaDB [(none)]> show variables like "%character%";show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
rows in set (0.01 sec)

+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
rows in set (0.00 sec)

MariaDB [(none)]>

未配置字符集前

```

```
MariaDB [(none)]> show variables like "%character%";show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
rows in set (0.00 sec)

+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_unicode_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
rows in set (0.00 sec)

MariaDB [(none)]>

配置字符集后
```

## 4. 远程链接mariadb数据库
mariadb默认是拒绝 root 远程登录的  
1). 关闭防火墙  
```SHELL
systemctl stop firewalld
```
在不关闭防火墙的情况下，允许某端口的外来链接。步骤如下，开启3306端口，重启防火墙  
```SHELL
firewall-cmd --query-port=3306/tcp  # 查看3306端口是否开启
firewall-cmd --zone=public --add-port=3306/tcp --permanent  # 开启3306端口
firewall-cmd --reload  # 重启防火墙
firewall-cmd --query-port=3306/tcp  # 查看3306端口是否开启
```

2). 先查看mysql数据库中的user表  
```
[root@mini ~]# mysql -u root -p  # 先通过本地链接进入数据库

MariaDB [(none)]> use mysql;

MariaDB [mysql]> select host, user from user;
+-----------+------+
| host      | user |
+-----------+------+
| 127.0.0.1 | root |
| ::1       | root |
| mini      | root |
+-----------+------+
rows in set (0.00 sec)
```

3). 将与主机名相等的字段改为 "%" ，我的主机名为mini，  
```
MariaDB [mysql]> update user set host='%' where host='mini';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [mysql]> select host, user from user;
+-----------+------+
| host      | user |
+-----------+------+
| %         | root |
| 127.0.0.1 | root |
| localhost | root |
+-----------+------+
rows in set (0.00 sec)
```

4). 刷新权限表，或重启mariadb服务，一下二选一即可
```SHELL
MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

systemctl restart mariadb
```
