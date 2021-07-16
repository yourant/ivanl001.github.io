[TOC]

## mysql安装(linux)

```shell
#检查mariadb是否已安装
yum list installed | grep mariadb 

#全部卸载
yum -y remove mariadb*

#下载并安装mysql的YUM源
wget -P /home/ec2-user/ http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

#安装mysql的YUM源：
rpm -ivh mysql57-community-release-el7-11.noarch.rpm

#检查mysql的YUM源是否安装成功：
yum repolist enabled | grep "mysql.*-community.*" 

#查看mysql版本
yum repolist all | grep mysql

#查看当前的启用的 MySQL 版本：
yum repolist enabled | grep mysql
# 安装MySQL
yum install mysql-community-server


#启动mysql服务(启动之后临时密码会写入日志中)：
systemctl start mysqld
# 查找密码
less /var/log/mysqld.log

# 设置开机启动？？？
# 查看是否是开机启动
systemctl list-unit-files | grep mysql
# 设置为开机启动
systemctl enable mysqld.service 

# 设置密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Zhangdf_123';


# 远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY 'Zhangdf_123' WITH GRANT OPTION;
flush privileges;
```





## mariadb(mysql)的安装

```shell
# 01, 使用yum直接安装MariaDB
yum -y install mariadb*

# 02, 开启服务，并设置开机启动
systemctl start mariadb.service
systemctl enable mariadb.service

# 03, 尝试登陆,刚开始是没有密码的，应该可以直接登陆成功
mysql

# 04, 如果成功之后，退出，设置密码
exit;
mysql_secure_installation;

# 05, 提示输入密码
# 注意：如果是刚装的，密码是空，直接enter即可，然后输入用户和密码，按照提示操作即可

# 06, 登陆即可
# root是你设置的用户名
mysql -u root -p *   

# 07， 设置外部访问
# 默认情况下，外部是不能直接连接我们的数据库的，需要更改设置,并刷新权限
# 这里是可以设置不同的用户名和不同的密码的哦，好强大的样子
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY ',.' WITH GRANT OPTION;
flush privileges;

# 08,  权限设置验证
# 可以从用户表中查看root访问权限
select host, user from mysql.user;
```



## mysql默认存储位置

```shell
/var/lib/mysql

# 也可以通过如下命令查看
ps -ef | grep mysql
# 输出结果大致如下：
mysql      1016      1  0 10:24 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql      1266   1016  0 10:24 ?        00:00:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
root      13995   1593  0 10:36 pts/0    00:00:00 grep --color=auto mysql
```

