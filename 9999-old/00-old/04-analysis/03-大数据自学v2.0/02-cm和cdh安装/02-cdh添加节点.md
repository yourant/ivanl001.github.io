http://www.sarame.cn/cm-cloudera-manager-ji-qun-tian-jia-shan-chu-zhu-ji/



### 添加新主机的方法举例（添加一台新的cdht6:10.10.140.56）

参考之前的安装教程，将cdht6进行相应的配置，包括hosts配置、ssh、防火墙、NTP时钟cdht1时间同步、卸载自带jdk等，并且把相应的CDH安装文件

```
    CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel,
    CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.sha,
    CDH-5.11.0-1.cdh5.11.0.p0.34-el6.parcel.torrent,
    manifest.json
```

拷贝到相应的目录/opt/cloudera/parcel-repo/

#### 下面开始新主机的添加：

```
#在cloudera-manager server所在服务器cdht1上执行如下命令：
scp -r /opt/cloudera-manager/cm-5.11.0 root@cdht6:/opt/cloudera-manager/

#在cdht6上创建用户以及用户组
useradd --system --home=/opt/cloudera-manager/cm-5.11.0/run/cloudera-scm-server --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

#然后再cdht6上执行如下命令,清除uuid等信息
rm -rf /opt/cloudera-manager/cm-5.12.1/lib/cloudera-scm-agent/*

#设置开机启动
cd /opt/cloudera-manager/cm-5.11.0/etc/init.d
chkconfig cloudera-scm-agent on 

#开启agent
./cloudera-scm-agent start
```

### 在CM管理界面进行操作：













另外注意同步==/usr/share/java/mysql-connector-java.jar==文件