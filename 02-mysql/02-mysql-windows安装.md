[toc]

### 1, 下载mysql安装包

https://dev.mysql.com/downloads/mysql/

![image-20200924115439761](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924115439761.png)



### 2, 解压缩后直接放在C:\Program Files目录下

![image-20200924115713174](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924115713174.png)





### 3, 找到bin目录位置，添加到环境变量中



![image-20200924115617532](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924115617532.png)



C:\Program Files\mysql-8.0.21-winx64\bin



![image-20200924115858566](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924115858566.png)



### 4, 初始化数据库：

```shell
mysqld --initialize --console
```



![image-20200924120119378](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924120119378.png)



<img src="/Users/ivanl001/Library/Application Support/typora-user-images/image-20200924120140945.png" alt="image-20200924120140945" style="zoom:50%;" />



执行完成后，会输出 root 用户的初始默认密码，如：

...
2018-04-20T02:35:05.464644Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: APWCY5ws&hjQ
...
APWCY5ws&hjQ 就是初始密码，后续登录需要用到，你也可以在登陆后修改密码。



### 5, 安装并启动：

```shell
mysqld install
net start mysql
```

![image-20200924120303234](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924120303234.png)

![image-20200924120319241](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924120319241.png)



### 6, 退出cmd重新登陆即可(记得先修改默认密码)

```shell
mysql -u root -p
# 输入4中获得的密码即可

# 登陆成功过后使用如下命令更改密码
alter user 'root'@'localhost' identified by 'ivanl001';
```

![image-20200924120426464](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20200924120426464.png)

