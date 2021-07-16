```shell
# 后台运行, 目前安装在nebula02上, 如果没有软链，那么直接使用绝对路径目录下的superset
nohup superset run -p 8089 -h 0.0.0.0 --with-threads &

nohup /usr/local/python3.7/bin/superset run -p 8089 -h 0.0.0.0 --with-threads &
```



[toc]



## 0, 参考文档：

http://superset.apache.org/

https://github.com/apache/incubator-superset

http://superset.apache.org/installation.html



## 1, 先安装python3.7版本

* 好像参考的是这个https://segmentfault.com/a/1190000018814995
  * 不要更改系统默认的python
  * 直接使用下载目录下的python安装使用superset即可

```shell
# 下载
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz

# 解压
tar -zxf Python-3.7.3.tgz

...看上面链接
```



## 2, 使用python3.7中pip安装superset

* http://superset.apache.org/installation.html

```shell
# 0, 先更新一下pip啥的, Put all the chances on your side by getting the very latest pip and setuptools libraries.:
/usr/local/python3.7/bin/pip install --upgrade setuptools pip

# 1, 安装superset, Install superset
/usr/local/python3.7/bin/pip install apache-superset

# 2, 安装后会在python3.7的bin目录下多一个superset软链, 使用该软链初始化数据库, Initialize the database
/usr/local/python3.7/bin/superset db upgrade

# 3, 设置管理员账号, Create an admin user (you will be prompted to set a username, first and last name before setting a password)
$ export FLASK_APP=superset
/usr/local/python3.7/bin/superset fab create-admin

# 4, 加载样例测试数据(无不需要可以不添加), Load some data to play with
superset load_examples

# 5, superset初始化, Create default roles and permissions
/usr/local/python3.7/bin/superset init
```



## 3, 配置文件

### 01, 配置环境变量PYTHONPATH

```shell
vim /etc/profile

export PYTHONPATH=/usr/local/python3.7/superset_config

```

### 02, 在目录内添加配置文件

```shell
vim /usr/local/python3.7/superset_config

# 添加如下配置文件
ROW_LIMIT = 50000
VIZ_ROW_LIMIT = 10000

SUPERSET_WEBSERVER_PORT = 8089
SUPERSET_WEBSERVER_ADDRESS = "0.0.0.0"

SUPERSET_WEBSERVER_TIMEOUT = 60
SUPERSET_DASHBOARD_POSITION_DATA_LIMIT = 65535
CUSTOM_SECURITY_MANAGER = None
SQLALCHEMY_TRACK_MODIFICATIONS = False

# ---------------------------------------------------
# Babel config for translations
# ---------------------------------------------------
# Setup default language
BABEL_DEFAULT_LOCALE = "en"
# Your application default translation path
BABEL_DEFAULT_FOLDER = "superset/translations"
# The allowed translation for you app
LANGUAGES = {
    "en": {"flag": "us", "name": "English"},
    "es": {"flag": "es", "name": "Spanish"},
    "it": {"flag": "it", "name": "Italian"},
    "fr": {"flag": "fr", "name": "French"},
    "zh": {"flag": "cn", "name": "Chinese"},
    "ja": {"flag": "jp", "name": "Japanese"},
    "de": {"flag": "de", "name": "German"},
    "pt": {"flag": "pt", "name": "Portuguese"},
    "pt_BR": {"flag": "br", "name": "Brazilian Portuguese"},
    "ru": {"flag": "ru", "name": "Russian"},
    "ko": {"flag": "kr", "name": "Korean"},
}
```





## 4, 启动superset

```shell
# To start a development web server on port 8088, use -p to bind to another port
nohup /usr/local/python3.7/bin/superset run -p 8089 -h 0.0.0.0 --with-threads &


druid://nebula02:8082/druid/pageview


from pageview order by __time desc
```





## 5, Q&A

```shell
# 在连接mysql的时候需要先安装python依赖, 可能需要重启superset
pip install mysqlclient

# 在连接hive的时候需要先安装python依赖, 可能需要重启superset
pip install pyhive

# 其他相关依赖参考：
http://superset.apache.org/installation.html
Database dependencies
```





## docker 版本

https://hub.docker.com/r/preset/superset/



```shell
docker pull preset/superset

docker run -d -p 8080:8080 --name superset preset/superset

docker exec -it superset superset fab create-admin \
               --username admin \
               --firstname Superset \
               --lastname Admin \
               --email admin@superset.com \
               --password admin
               
               
docker exec -it superset superset db upgrade

docker exec -it superset superset load_examples

docker exec -it superset superset init
```





# How to use this image

## Start a superset instance on port 8080

```
$ docker run -d -p 8080:8080 --name superset preset/superset
```

## Initialize a local Superset Instance

**With your local superset container already running...**

1. Setup your local admin account

   ```
    $ docker exec -it superset superset fab create-admin \
                  --username admin \
                  --firstname Superset \
                  --lastname Admin \
                  --email admin@superset.com \
                  --password admin
   ```

2. Migrate local DB to latest

   ```
    $ docker exec -it superset superset db upgrade
   ```

3. Load Examples

   ```
    $ docker exec -it superset superset load_examples
   ```

4. Setup roles

   ```
    $ docker exec -it superset superset init
   ```

5. Login and take a look -- navigate to http://localhost:8080/login/ -- u/p: [admin/admin]













https://preset.io/blog/2020-05-11-getting-started-installing-superset/



```
docker-compose up
docker-compose up --build

//国际化支持
pybabel compile -d translations

```