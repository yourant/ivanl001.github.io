[toc]

<extoc></extoc>



## 1, 空白主机处理



### 1.1, 服务器基本情况查看

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


# 1、查看 CPU 物理个数
grep 'physical id' /proc/cpuinfo | sort -u | wc -l

# 2、查看 CPU 核心数量
grep 'core id' /proc/cpuinfo | sort -u | wc -l

# 3、查看 CPU 线程数
grep 'processor' /proc/cpuinfo | sort -u | wc -l

# 4、查看 CPU  型号
dmidecode -s processor-version

# 5、查看 CPU 的详细信息：
cat /proc/cpuinfo
```



### 1.2, 安装需要的软件

> yum install vim net-tools.x86_64 nc telnet rsync ntp ntpdate -y



### 1.3, 修改hostname

> vim /etc/hostname

```java
把里面的内容更换成想要的主机名后reboot即可
```

> 如果不想重启，操作上述内容后通过命令：hostname nebula09可以临时修改hostname

```shell
hostname nebula09
```



### 1.4, ssh免密登陆

- 这步最好在设置hostname之后，因为公钥中会保存hostname相关的内容

> ssh-keygen

```java
一路回车enter即可，完成后，/root/.ssh目录下会有id_rsa和id_rsa.pub两个文件
所有需要互相免密登陆的id_rsa.pub中的内容放到一个文件authorized_keys文件中，并把authorized_keys分发到各个服务器的/root/.ssh/目录下即可
```

> cd .ssh



scp -P 38022 /root/.ssh/authorized_keys root@nebula09:/root/.ssh/



### 1.5, 自定义工具imcall.sh, imrsync.sh

* 这里留意一下，如果环境变量中没有/usr/local/bin的话， 直接在下面添加可执行文件并不能直接调用哈

==/usr/local/bin目录如果没在环境变量中的话， 需要进行软链才能直接调用命令==

`⚠️注意：imcall jps不能使用的时候是因为环境变量的原因，一般ssh过去使用的环境变量是.bashrc文件中的，所以把profile的内容更换到.bashrc文件中即可`

```shell
cat /etc/profile >> ~/.bashrc
```



> vim /usr/local/bin/imrsync.sh
>
> > 添加下面的shell脚本后，修改执行权限
>
> chmod u+x /usr/local/bin/imrsync.sh

```shell
#! /bin/bash

# 脚本作用：同步文件或者文件夹,可以输入全路径，也可以输入相对路径或者当前的某个文件名
if(($#!=1)); then echo '请输入正确的需要同步的文件夹或文件的路径';exit;fi;
echo;
path=$@;

relativeDirPath=`dirname $path`;#这里取出来有时候是.,也就是当前目录
cd $relativeDirPath;
dirPath=`pwd`;

basePath=`basename $path`;
fullPath=$dirPath/$basePath;

echo '--------------------------------同步源目录开始:'$fullPath'------------------------------------';
for((i=1;i<4;i++));
do
rsync -avl $@ root@centos0$i:$fullPath;
done;
echo '--------------------------------同步源目录结束:'$dirPath/$basePath'------------------------------------';

echo;
```

> vim /usr/local/bin/imcall.sh
>
> > 添加下面的shell脚本后，修改执行权限
>
> chmod u+x /usr/local/bin/imcall.sh

- 注意：如果又时候不管用，类似imcall.sh jps这样的，是因为ssh上另外一台服务器的时候用的是~/.bashrc文件的，需要把java的环境变量也加入一份到~/.bashrc文件中去就可以了哈

```shell
#! /bin/bash

#if((n!=1)); then echo '请输入正确命令';exit;fi;#这里不能判断一个参数，不然ls -a这种就会有问题
echo;#打印一行空格

echo '---------------------------------执行命令开始:'$@'-----------------------------------';
echo;#打印一行空格
for((i=1;i<4;i++))
do
echo '---------------'centos0$i'--------------';
ssh centos0$i "$@";
done;
echo;
echo '---------------------------------执行命令结束:'$@'-----------------------------------';

```



### 1.6, 修改hosts文件

> vim /etc/hosts

```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.147.101 centos01
192.168.147.102 centos02
192.168.147.103 centos03
```





### 1.7, jdk的安装

* 这里也可以直接拷贝

#### 7.1, 下载jdk-8u211-linux-x64.tar.gz

#### 7.2, 解压缩至/usr/local/目录下

> tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/

#### 7.3, 创建软链接

> ln -s jdk1.8.0_211/ jdk

#### 7.4, 添加环境变量并更新

> vim /etc/profile
>
> > 添加如下内容后source
> >
> > ```shell
> > export JAVA_HOME=/usr/local/jdk
> > export PATH=$PATH:$JAVA_HOME/bin
> > ```
>
> source /etc/profile

#### 7.5, 验证

> java



### 1.8, 关闭防火墙

> 查看防火墙状态

`systemctl status firewalld.service`

> 关闭防火墙

`systemctl stop firewalld.service`

> 打开防火墙

`systemctl start firewalld.service`

> 禁用防火墙

`systemctl disable firewalld.service`

> 启用防火墙

`systemctl enable firewalld.service`



### 1.9, 时间同步并修改时区

#### 1.安装ntpdate工具

```shell
yum -y install ntp ntpdate
```

#### 2.设置系统时间与网络时间同步

```shell
ntpdate cn.pool.ntp.org
```

#### 3.将系统时间写入硬件时间

```shell
hwclock --systohc
```

#### 4.查看系统时间

```shell
timedatectl
```

#### 5.更改时区

```shell
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 有提示的时候输入yes即可

# 更改为洛杉矶时区
cp /usr/share/zoneinfo/America/Los_Angeles /etc/localtime

# 观察，需要也修改这个
timedatectl set-timezone America/Los_Angeles
```



### 2.0, 关闭selinu

> vim /etc/sysconfig/selinux
>
> setenforce 0

```
SELINUX=disabled
```

* 重启启动服务器reboot生效

* 查看状态：

  > /usr/sbin/sestatus -v



### 2.1, 安装依赖包

```shell
yum install chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb httpd mod_ssl -y

# 下面这些先不装
yum install krb5-devel cyrus-sasl-gssapi cyrus-sasl-deve libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel -y
```





## 2, 添加并开启cm-agent

### 2.1, 拷贝对应应用

```shell
# 这个是cm的管理文件，必须拷贝
rsync -avl -e 'ssh -p 38022' /opt/cloudera-manager/ root@nebula09:/opt/cloudera-manager;
# 这个是对应的parcels文件(如果不copy后续会自动下载，速度会比较慢)
rsync -avl -e 'ssh -p 38022' /opt/cloudera/ root@nebula09:/opt/cloudera;
```



### 2.2, 删除对应的uuid信息等

* 注意：这一步如果是拷贝其他机器上文件，必须要做，否则可能会启动一些应用，会影响现有集群

```shell
#然后再cdht6上执行如下命令,清除uuid等信息
rm -rf /opt/cloudera-manager/cm-5.12.1/lib/cloudera-scm-agent/*


#设置开机启动(我这里没有开启这个哈)
cd /opt/cloudera-manager/cm-5.12.1/etc/init.d
chkconfig cloudera-scm-agent on 

```



### 2.3, 开启agent

```shell
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start

# 可以通过如下地址查看日志
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log
```



## 3, 添加主机操作

* 参考：http://www.sarame.cn/cm-cloudera-manager-ji-qun-tian-jia-shan-chu-zhu-ji/

```shell
# 页面添加主机比较简单，参考如上链接
```



## 4, 善后工作

### 4.1, 同步sql的连接器

```shell
# imrsync.sh脚本参考1.5, 自定义工具imcall.sh, imrsync.sh
imrsync.sh /usr/share/java/mysql-connector-java.jar
```



### 4.2, 把新添加机器加入hive集群

* 其实就是在新加机器上添加hive的gateway服务不然无法连接现有集群的数据仓库

![image-20210508195121758](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20210508195121758.png)

### 4.3, 添加其他需要节点

```shell
#  这个按需添加即可
```



