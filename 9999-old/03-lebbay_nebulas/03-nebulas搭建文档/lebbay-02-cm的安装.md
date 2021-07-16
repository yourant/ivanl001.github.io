```shell
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-server/cloudera-scm-server.log
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log
```



```shell
# !!! 注意先切换到root用户下
sudo su
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server start
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start
# imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start

# !!! 注意先切换到root用户下
# 先关闭agent，再关闭server
sudo su
imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server stop
```



[toc]



## 01-cm和cdh环境搭建

> cm是Cloudera Manager，cdh是Cloudera Distributed Hadoop。前者是管理后者的一个平台，后者是Hadoop的一个发行版本。

```shell
# 首先还是要在root下执行
sudo su
```



### 1, 安装依赖包

```shell
yum install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb httpd mod_ssl -y

# 下面这些先不装(好像不是必须的)
yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel openldap-devel python-devel python-simplejson sqlite-devel -y
```



### 2, cm下载

#### 01, cm包

```shell
wget http://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.12.1_x86_64.tar.gz
```

#### 02, parcel文件

```shell
wget http://archive.cloudera.com/cdh5/parcels/5.12.1/CDH-5.12.1-1.cdh5.12.1.p0.3-el7.parcel
```

#### 03, sha文件

```shell
wget http://archive.cloudera.com/cdh5/parcels/5.12.1/CDH-5.12.1-1.cdh5.12.1.p0.3-el7.parcel.sha1
# 需要把文件名末尾的1给去掉
mv CDH-5.12.1-1.cdh5.12.1.p0.3-el7.parcel.sha1 CDH-5.12.1-1.cdh5.12.1.p0.3-el7.parcel.sha
```

#### 04, manifest文件

```shell
wget http://archive.cloudera.com/cdh5/parcels/5.12.1/manifest.json
```



### 3, cm的解压缩和安装

#### 01,  解压文件，并分发到每个机器上

```shell
# 创建文件夹
mkdir -p /opt/cloudera-manager/
# 解压文件
tar -zxvf cloudera-manager-centos7-cm5.12.1_x86_64.tar.gz -C /opt/cloudera-manager
# 分发
imrsync.sh /opt/cloudera-manager/
```

#### 02, 添加用户cm用户

```shell
sudo useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

# 查看用户是否创建ok
cat /etc/passwd
cat /etc/passwd 可以查看所有用户的列表
w 可以查看当前活跃的用户列表
cat /etc/group 查看用户组

groups 查看当前登录用户的组内成员
groups gliethttp 查看gliethttp用户所在的组,以及组内成员
whoami 查看当前登录用户名

一个简明的layout命令

cat /etc/passwd|grep -v nologin|grep -v halt|grep -v shutdown|awk -F":" '{ print $1"|"$3"|"$4 }'|more
```

#### 03,  修改配置文件

```shell
vim /opt/cloudera-manager/cm-5.12.1/etc/cloudera-scm-agent/config.ini

# Hostname of the CM server.
server_host=nebula01
```

#### 04,  jdbc的连接依赖

> 需要在有mysql的机器上
>
> - 下载mysql-connector-java-5.1.26-bin.jar
> - 把mysql-connector-java-5.1.26-bin.jar拷贝到/usr/share/java/目录下，并更改名字为：mysql-connector-java.jar

```shell
mkdir -p /usr/share/java
mv mysql-connector-java.jar /usr/share/java
```

#### 05, 配置CM Server数据库

```shell
grant all on *.* to 'cdh'@'%' identified by 'cdh' with grant option;

# 要刷新权限
flush privileges;

# 然后到/opt/cloudera-manager/cm-5.7.2/share/cmf/schema目录下，执行如下命令：
/opt/cloudera-manager/cm-5.7.2/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm

/opt/cloudera-manager/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh mysql cdh -h nebula01 -u cdh -p cdh --scm-host nebula01 scm scm scm

-h database host
-u database username
-p database password
--scm-host scm server hostname

/opt/cloudera-manager/cm-5.12.1/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm
# 注意ccscs限的时候就是cdh
/opt/cloudera-manager/cm-5.16.2/share/cmf/schema/scm_prepare_database.sh mysql cdh -h centos01 -u cdh -p cdh --scm-host centos01 scm scm scm
```

#### 06, 创建Parcel目录,更改权限等

> *也就是各种组件的仓库*

- server节点

  ```shell
  sudo mkdir -p /opt/cloudera/parcel-repo
  sudo chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
  ```

- Agent节点

  ```shell
  sudo mkdir -p /opt/cloudera/parcels
  sudo chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
  ```

#### 07, 制作CDH本地源

* 如下操作只在server节点即可

* *把上面下载parcel文件，sha文件和manifest.json文件放到server节点的/opt/cloudera/parcel-repo下*



### 4, cm的启动

#### 01, 在Server节点上开启server

* 成功后，jps会多处一个main进程，日志中无报错

```shell
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server start
# 日志位置
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-server/cloudera-scm-server.log 
```

#### 02, 在agent节点上开启agent

```shell
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start
# 日志位置
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log
```

#### 03, 页面验证

* 只有配置了hosts才能直接访问

* http://nebula01:7180/

