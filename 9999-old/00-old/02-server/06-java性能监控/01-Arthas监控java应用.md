[toc]

https://www.cnblogs.com/javastack/p/11273902.html





## 1, 安装

> 参考：https://alibaba.github.io/arthas/install-detail.html

```shell
# 下载arthas-boot.jar包
curl -O https://alibaba.github.io/arthas/arthas-boot.jar
# 启动arthas-boot.jar包就相当于启动了arthas应用了
java -jar arthas-boot.jar


# 当然也可以使用shell脚本
curl -L https://alibaba.github.io/arthas/install.sh | sh
# 这样就可以使用该命令了，和上面的java -jar arthas-boot.jar作用一样、
./as.sh
```



页面验证：(这是一个页面交互程序)

http://127.0.0.1:3658/



## 2, 使用

> 参考：https://alibaba.github.io/arthas/quick-start.html

```shell
# 通过上面进行命令行之后， 会自动打印本机上的java应用的pid，根据提示输入对应pid的序列号即可进行该应用的监控
1

# 然后主要命令：

dashboard

#  监控线程
thread

# 这个会反编译.class为源码
jad com.lebbay.nebulasevent.controller.IMEventController
```





