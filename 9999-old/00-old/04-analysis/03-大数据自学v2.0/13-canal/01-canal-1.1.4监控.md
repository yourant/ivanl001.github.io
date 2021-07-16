[TOC]

> 参考内容：https://github.com/alibaba/canal/wiki/Prometheus-QuickStart
>
> ## Quick start
>
> 1. 安装并部署对应平台的prometheus，参见[官方guide](https://prometheus.io/docs/introduction/first_steps/)
> 2. 配置prometheus.yml，添加canal的job，示例：
>
> ```yaml
>   - job_name: 'canal'
>     static_configs:
>     - targets: ['localhost:11112'] //端口配置即为canal.properties中的canal.metrics.pull.port
> ```
>
> 1. 启动prometheus与canal server
> 2. 安装与部署[grafana](http://docs.grafana.org/)，推荐使用新版本(5.2)。
> 3. 启动grafana-server，使用用户admin与密码admin登录localhost:3000 (默认配置下)。
> 4. 配置[prometheus datasource](http://docs.grafana.org/features/datasources/prometheus/#adding-the-data-source-to-grafana).
> 5. 导入模板([canal/conf/metrics/Canal_instances_tmpl.json](https://raw.githubusercontent.com/alibaba/canal/master/deployer/src/main/resources/metrics/Canal_instances_tmpl.json))，参考[这里](http://docs.grafana.org/reference/export_import/#importing-a-dashboard)。
> 6. 进入dashboard 'Canal instances', 在'datasource'下拉框中选择刚才配置的prometheus datasource, 然后'destination'下拉框中就可以切换instance了(如果没出现instances列表就刷新下页面), just enjoy it.



## 1, canal状态监控

* 参考内容：https://github.com/alibaba/canal/wiki/Canal-Admin-Guide



### 01, 下载canal.admin

```shell
wget https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.admin-1.1.4.tar.gz
```



### 02, 解压缩

```shell
mkdir /tmp/canal-admin

tar -zxvf canal.admin-1.1.4.tar.gz -C /usr/local/canal.admin-1.1.4/
```



### 03, 修改配置conf/application.yml

```yaml
vim /usr/local/canal-admin/conf/application.yml

# 我就按照默认，如下，其实这里不需要修改
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  address: 127.0.0.1:3306
  database: canal_manager
  username: canal
  password: canal
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin
  adminPasswd: admin
```

### 04, 初始化元数据库

* 初始化SQL脚本里会默认创建canal_manager的数据库，建议使用root等有超级权限的账号进行初始化 b. canal_manager.sql默认会在conf目录下，也可以通过链接下载 [canal_manager.sql](https://raw.githubusercontent.com/alibaba/canal/master/canal-admin/canal-admin-server/src/main/resources/canal_manager.sql)

```shell
# 使用root账号登陆mysql
mysql -u root -p

# 导入初始化SQL
source /usr/local/canal-admin/conf/canal_manager.sql
```



### 05，启动canal.admin

```shell
# 开启
/usr/local/canal.admin-1.1.4/bin/startup.sh

# 关闭
/usr/local/canal.admin-1.1.4/bin/stop.sh
```



### 06，页面验证, 默认端口: 8089

* http://192.168.215.103:8089

```mysql
admin
123456(默认)
```





## 2, Prometheus性能监控

*　参考内容：https://github.com/alibaba/canal/wiki/Prometheus-QuickStart

### 01，prometheus下载解压

* 我使用的时候下载的是最新版本2.12.0

*　https://github.com/prometheus/prometheus/releases/download/v2.12.0/prometheus-2.12.0.linux-amd64.tar.gz
*　下载后解压缩到/usr/local目录下

### 02, 配置prometheus.yml

*　配置job

```yaml
  # 我使用HA模式，有两个，所以配置了两个job
  - job_name: 'canal101'
    static_configs:
    - targets: ['centos01:11112']

  - job_name: 'canal102'
    static_configs:
    - targets: ['centos02:11112']                      
```



### 03, 启动prometheus

```shell
# jps查不出来，应该不是java应用
/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml > /dev/null 2>&1 &

# 如果需要停止
ps -ef | grep prometheus
kill -9 pid
```



### 04, 页面验证, 默认端口: 9090

* [http://centos01:9090](http://centos01:9090/)





## 3，grafana性能监控显示

### 01, 下载grafana解压等

* canal官方建议使用5.2.0版本，所以我这里下载5.2.0

* https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.2.0.linux-amd64.tar.gz
* 解压到/usr/local目录下



### 02, 不需要配置，直接启动

```shell
/usr/local/grafana/bin/grafana-server web > /dev/null 2>&1 &

# 如果需要停止
ps -ef | grep grafana
kill -9 pid
```



### 03，页面验证, 默认端口：3000

* http://centos01:3000/



### 04, 添加Prometheus数据源

#### Adding the data source to Grafana[¶](https://grafana.com/docs/features/datasources/prometheus/#adding-the-data-source-to-grafana)

1. Open the side menu by clicking the Grafana icon in the top header.
2. In the side menu under the `Dashboards` link you should find a link named `Data Sources`.
3. Click the `+ Add data source` button in the top header.
4. Select `Prometheus` from the *Type* dropdown.



### 05，导入数据模板

[canal/conf/metrics/Canal_instances_tmpl.json](https://raw.githubusercontent.com/alibaba/canal/master/deployer/src/main/resources/metrics/Canal_instances_tmpl.json)，参考[这里](http://docs.grafana.org/reference/export_import/#importing-a-dashboard)。



### 06，选择instance即可开始监控

*　进入dashboard 'Canal instances', 在'datasource'下拉框中选择刚才配置的prometheus datasource, 然后'destination'下拉框中就可以切换instance了(如果没出现instances列表就刷新下页面), just enjoy it.
*　