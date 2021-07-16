## 1, 新增并设置远程调试服务器

![1571363462278](assets\1571363462278.png)



![1571363519903](assets\1571363519903.png)



## 2, 现在远程服务器上启动jar包，命令如下

* 部分参数是从刚才的配置页面中取出来的

```shell
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8089 -jar report-0.0.1-SNAPSHOT.jar
```



## 3, debug远程服务

![1571363654895](assets\1571363654895.png)