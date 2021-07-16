[TOC]



## 服务器搭建基础命令

```shell
# 服务器的基本情况查看

# 查看内存使用情况
free -m
 
# 查看cpu使用情况进程运行情况
top
 
# 查看磁盘以及分区情况
df -h 
 
# 查看网络情况
ifconfig

# 查看端口是否可以访问,比如某个程序的端口是否正常
telnet 192.168.147.101 7180
# 如何提示如下，说明端口正常可以连接
$> Trying 192.168.147.101...
$> Connected to 192.168.147.101.
$> Escape character is '^]'.


# 查看端口使用情况
# 1.方法一
lsof -i:80
# 2.方法二
netstat -anop | grep 80
a: -a或--all 显示所有连线中的Socket
n: -n或--numeric 直接使用IP地址，而不通过域名服务器
o: -o或--timers 显示计时器。
p: -p或--programs 显示正在使用Socket的程序识别码和程序名称。


# 3.方法三
ps -au | grep 80
```



## 服务器信息: 7台

| hostname | mem             | disk          | core |      |
| -------- | --------------- | ------------- | ---- | ---- |
| nebula01 | 64G             | 300G          | 8    |      |
| nebula02 | 64G             | 300G          | 8    |      |
| nebula03 | 32G             | 300G          | 8    |      |
| nebula04 | 32G             | 300G          | 8    |      |
| nebula05 | 32G             | 300G          | 8    |      |
| nebula06 | 32G             | 300G          | 8    |      |
| nebula07 | 32G             | 300G          | 8    |      |
|          | free -h查看内存 | df -h查看磁盘 |      |      |



## 0, 注意以下命令若无说明，均在root下执行

```shell
# 切换到root用户
sudo su
```



## 1, 检查网络，系统，内存，磁盘等信息

```shell
# 查看网络是否正常
ping www.google.com
ping www.baidu.com
# 查看内存空间
free -h
# 查看磁盘空间
df -h
# 查看系统
uname
uname -a
# 查看系统版本
cat /etc/redhat-release
rpm -q centos-release
# 查看内核
uname  -r
```



## 2, 首先安装需要的基础软件

```shell
# 这里针对的是原生的mini系统
yum install vim net-tools.x86_64 nc telnet rsync ntp ntpdate wget -y
```



## 3, 修改hostname

```shell
# 临时生效
hostname nebula01

# 永久更改
# 把里面的内容更换成想要的主机名后reboot即可
vim /etc/hostname

nebula01
```



## 4, 修改hosts文件

```shell
vim /etc/hosts
```

* 内容更改如下

```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# 添加如下内容
# 内网hosts，用在内网内的系统互相关联
172.31.29.76  nebula01
172.31.70.190 nebula02
172.31.12.253 nebula03
172.31.34.134 nebula04
172.31.85.249 nebula05
172.31.19.140 nebula06
172.31.77.55  nebula07
```



## 5, ssh免密登陆

- 这步最好在设置hostname之后，因为公钥中会保存hostname相关的内容

```shell
# 注意不同的用户， 我这里在root和centos用户下都执行了
# root用户下ssh-keygen
sudo su
ssh-keygen
# 退出root用户
exit
ssh-keygen
```

* 一路回车enter即可，完成后，~/.ssh目录下会有id_rsa和id_rsa.pub两个文件
* 所有需要互相免密登陆的id_rsa.pub中的内容放到一个文件authorized_keys文件中，并把authorized_keys分发到各个服务器的~/.ssh/目录下即可

```shell
# root用户
scp -P 38022 /root/.ssh/authorized_keys nebula02:/root/.ssh/authorized_keys
# centos用户
scp -P 38022 /home/centos/.ssh/authorized_keys nebula02:/home/centos/.ssh/authorized_keys
```



## 6, 自定义工具imcall.sh, imrsync.sh

### 6.1, imrsync.sh

```shell
vim /usr/local/bin/imrsync.sh
# 保存后更改权限
chmod u+x /usr/local/bin/imrsync.sh
```

* 内容如下：

```shell
#! /bin/bash

# 脚本作用：同步文件或者文件夹,可以输入全路径，也可以输入相对路径或者当前的某个文件名
if(($#!=1)); then echo '请输入正确的需要同步的文件夹或文件的路径';exit;fi;
echo;
path=$@;

relativeDirPath=`dirname $path`; #这里取出来有时候是.,也就是当前目录
cd $relativeDirPath;
dirPath=`pwd`;

basePath=`basename $path`;
fullPath=$dirPath/$basePath;

echo '--------------------------------同步源目录开始:'$fullPath'------------------------------------';
for((i=1;i<8;i++));
do
echo '---------------'nebula0$i'--------------';
rsync -avl -e 'ssh -p 38022' $@ root@nebula0$i:$fullPath;
done;
echo '--------------------------------同步源目录结束:'$dirPath/$basePath'------------------------------------';

echo;
```



### 6.2, imcall.sh

> 注意：如果又时候不管用，类似imcall.sh jps这样的，是因为ssh上另外一台服务器的时候用的是~/.bashrc文件的，需要把java的环境变量也加入一份到~/.bashrc文件中去就可以了哈

```shell
vim /usr/local/bin/imcall.sh
# 保存后更改权限
chmod u+x /usr/local/bin/imcall.sh
```

```shell
#! /bin/bash

#if((n!=1)); then echo '请输入正确命令';exit;fi;#这里不能判断一个参数，不然ls -a这种就会有问题
echo;#打印一行空格

echo '---------------------------------执行命令开始:'$@'-----------------------------------';
echo;#打印一行空格
for((i=1;i<8;i++))
do
echo '---------------'nebula0$i'--------------';
ssh nebula0$i -p 38022 "$@";
done;
echo;
echo '---------------------------------执行命令结束:'$@'-----------------------------------';
```



### 6.3, root下无法执行，软链到/usr/bin下

```shell
ln -s /usr/local/bin/imrsync.sh /usr/bin/imrsync.sh
ln -s /usr/local/bin/imcall.sh /usr/bin/imcall.sh
```





## 7, jdk的安装

### 7.1, 下载jdk-8u211-linux-x64.tar.gz

```shell
# 没找到wget的下载地址
wget ...
```



### 7.2, 解压缩至/usr/local/目录下

```shell
tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/
```

### 7.3, 创建软链接

```shell
ln -s jdk1.8.0_211/ jdk
```

### 7.4, 添加环境变量并更新

```shell
vim /etc/profile

# 更新使生效
source /etc/profile
```

```shell
# 添加如下内容后source
export JAVA_HOME=/usr/local/jdk/
export PATH=$PATH:$JAVA_HOME/bin
```



### 7.5, 验证

```shell
java
java -version
```



### 7.6, 在 ~/.bashrc下也添加java权限

* ⚠️注意：imcall jps不能使用的时候是因为环境变量的原因，一般ssh过去使用的环境变量是.bashrc文件中的，所以把profile的内容更换到.bashrc文件中即可

* 最好不同的用户下的.bashrc都添加一下

```shell
cat /etc/profile >> ~/.bashrc
```



## 8, 关闭防火墙和selinu

### 01, 关闭防火墙

> 我们的系统中好像没有这个，所以就不需要关闭了，应该是用的安全组

```shell
# 查看防火墙状态
systemctl status firewalld.service

# 关闭防火墙
systemctl stop firewalld.service

# 打开防火墙
systemctl start firewalld.service

# 禁用防火墙
systemctl disable firewalld.service

# 启用防火墙
systemctl enable firewalld.service
```

### 02, 关闭selinu

```shell
# 临时生效
setenforce 0
# 查看临时更改是否生效
getenforce

# 永久更改(需要重启)
vim /etc/sysconfig/selinux
# 修改如下属性
SELINUX=disabled
# 查看永久更改是否生效
/usr/sbin/sestatus -v
```



## 9, mysql安装和设置

### 01, mysql安装

> 我们这里分别在nebula01,nebula02上安装了mysql

```shell
#检查mariadb是否已安装
yum list installed | grep mariadb 

#全部卸载
yum -y remove mariadb*

#下载并安装mysql的YUM源
wget -P /home/centos/nebulas/ http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

#安装mysql的YUM源：
rpm -ivh mysql57-community-release-el7-11.noarch.rpm

#检查mysql的YUM源是否安装成功：
yum repolist enabled | grep "mysql.*-community.*" 

#查看mysql版本
yum repolist all | grep mysql

#查看当前的启用的 MySQL 版本：
yum repolist enabled | grep mysql
# 安装MySQL
yum install mysql-community-server -y


#启动mysql服务(启动之后临时密码会写入日志中)：
systemctl start mysqld
# 查找密码
less /var/log/mysqld.log


# 查看是否是开机启动
systemctl list-unit-files | grep mysql
# 设置为开机启动
systemctl enable mysqld.service 


# 设置密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '***********';


# 远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '***********' WITH GRANT OPTION;
flush privileges;



GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'ivanl001' WITH GRANT OPTION;
flush privileges;
```



### 02, mysql开启主从复制

* 参考官网内容, 版本5.7，如果是其他版本，请参考官网其他版本的内容
* https://dev.mysql.com/doc/refman/5.7/en/replication-howto.html

```shell
# 1.1, 修改主节点的配置文件,开启binlog日志
sudo vim /etc/my.cnf
# 添加如下两行
# [mysqld]
log-bin=mysql-bin
server-id=1

# 1.2, 重启mysql
sudo systemctl restart mysqld

# 2, 在主节点上给从节点赋予权限
CREATE USER 'repl'@'172.31.12.229' IDENTIFIED BY 'yfrmJ1Vt9_NyCfi7';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'172.31.12.229';
flush privileges;


CREATE USER 'repl'@'nebula02' IDENTIFIED BY 'yfrmJ1Vt9_NyCfi7';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'nebula02';
flush privileges;


# 3, 锁定主节点，从主节点上获取binlog日志位置
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;


# 4，先锁定，同步历史数据
# 在主节点使用如下命令把历史数据生成文件以方便后续导入
mysqldump -u root -p --all-databases --master-data > dbdump.db


# 5，同步数据之后，设置slave节点配置文件
sudo vim /etc/my.cnf
# 添加如下一行
# [mysqld]
server-id=2
# 重启mysql
sudo systemctl restart mysqld

# 6，历史数据执行
# 在从节点执行dbdump.db文件即可
mysql -u root -p < dbdump.db


# 7, 打开主节点锁定
UNLOCK TABLES;


# 8，在从节点上设置主节点信息
# master_log_file是刚才show master status时候的那个文件或者你想要启动的那个文件
change master to master_host='nebula01',master_user='repl',master_password='yfrmJ1Vt9_NyCfi7',master_log_file='mysql-bin.000001',master_log_pos=1018;

#启动从服务器复制功能
start slave;
stop  slave;

# 验证从节点的状态
show slave status \G;

# 只有下面两个属性全部为Yes的时候才算成功
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```



## 10, mariadb安装

> 我们系统中安装的是mysql，所以这里就不需要安装了

*现在已经用MariaDB完全取代msyql，而是使用方法完全一致，并且安装更方便一些，安装方法如下：*

- 01， 使用yum直接安装MariaDB

  > yum -y install mariadb*

- 02, 开启服务，并设置开机启动

  > systemctl start mariadb.service
  > systemctl enable mariadb.service

- 03, 尝试登陆,刚开始是没有密码的，应该可以直接登陆成功

  > mysql

- 04，如果成功之后，退出，设置密码

  > exit;
  >
  > mysql_secure_installation;

- 05，提示输入密码，如果是刚装的，密码是空，直接enter即可，然后输入用户和密码，按照提示操作即可

- 06, 都成功之后，可以按照正常流程进行登陆即可

  > mysql -u root -p *****   //root是你设置的用户名

- 07， 默认情况下，外部是不能直接连接我们的数据库的，需要更改设置,并刷新权限

  - 这里是可以设置不同的用户名和不同的密码的哦，好强大的样子

  ```mysql
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '',.' WITH GRANT OPTION;
  flush privileges;
  ```

  > GRANT ALL PRIVILEGES ON \*.\* TO 'root'@'%'IDENTIFIED BY ',.' WITH GRANT OPTION;
>
  > flush privileges;

- 08, 可以从用户表中查看root访问权限

  > select host, user from mysql.user;
  
  

## 11, 时间同步并修改时区

> 系统中已经自动同步时间了，所以时间同步就不需要做了，但是需要修改默认的UTC时区为洛杉矶时区

```shell
# 查看时区
timedatectl list-timezones
# 这个和下面的那句应该是等价的
# timedatectl set-timezone America/Los_Angeles


# 首先更改centos时区
sudo ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime


# 然后jre时区是从/etc/sysconfig/clock文件中读取，更改
sudo vim /etc/sysconfig/clock

# ZONE="UTC"
# UTC=true

ZONE="America/Los_Angeles"
UTC=false
ARC=false
```

