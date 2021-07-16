https://clickhouse.tech/docs/zh/getting-started/install/





### `RPM`安装包[ ](https://clickhouse.tech/docs/zh/getting-started/install/#from-rpm-packages)

推荐使用CentOS、RedHat和所有其他基于rpm的Linux发行版的官方预编译`rpm`包。

首先，您需要添加官方存储库：

```
sudo yum install yum-utils
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64
```

如果您想使用最新的版本，请用`testing`替代`stable`(我们只推荐您用于测试环境)。`prestable`有时也可用。

然后运行命令安装：

```
sudo yum install clickhouse-server clickhouse-client
```

你也可以从这里手动下载安装包：[下载](https://repo.clickhouse.tech/rpm/stable/x86_64)。



## 启动[¶](https://clickhouse.tech/docs/zh/getting-started/install/#qi-dong)

```shell
# 配置允许远程连接
sudo vim /etc/clickhouse-server/config.xml

# 打开下面这一行
<listen_host>::</listen_host>
```





```shell
# 启动服务
systemctl start clickhouse-server.service
# 关闭开机启动
systemctl disable clickhouse-server.service

# 查看服务状态
systemctl status clickhouse-server.service
systemctl status clickhouse-server

# 查看启动端口
netstat -anop | grep 9000


# 远程连接(远程连接先修改配置允许远程连接)
clickhouse-client

# 多行操作
clickhouse-client -m
```

