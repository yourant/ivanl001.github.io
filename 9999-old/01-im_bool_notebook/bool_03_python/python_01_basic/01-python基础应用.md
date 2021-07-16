
[TOC]

## 1, python的文件读取及复制

```python
#
# author      : ivanl001
# creator     : 2018-12-02 19:26
# description : python文件读取和复制等
#

import sys

# 1, 一次性读
print("# 1, 一次性读")
f = open("./ivanl001.txt")
lines = f.readlines()
for line in lines:
    # 这个意思是打印但是不换行
    print(line, end='')

# 2, 每次读取一行
print()
print("2, 每次读取一行")
f = open("./ivanl001.txt")
while True:
    line = f.readline()
    if line != '':
        print(line, end="")
    else:
        # 这里就是换个行
        print()
        break

# 3,字节读取，文件拷贝等
print("# 3,字节读取，文件拷贝等")
input01 = open("./test.jpg", "rb")
output01 = open("./test-copy.jpg", "wb")

print("拷贝开始")
while True:
    print("---0000")
    # 这里是设置缓存，一次缓存多少字节
    data = input01.read(1024*1024)  # 这里是1024*1024byte，也就是1M
    print(len(data))
    if len(data) == 0:
        break
    else:
        output01.write(data)

input01.close()
output01.close()
print("拷贝结束")

# 这里是退出执行
sys.exit()

```



## 2, python连接mysql

* 关于pymysql和mysqldb
* mysqldb是python2版本提供的数据库连接的工具
* 在python3中已经没有这个了，在python3中用pymysql进行替换即可
* PyMySQL 是在 Python3.x 版本中用于连接 MySQL 服务器的一个库，Python2中则使用mysqldb。

> 数据库连接池连接数据库



### 01, 首先创建一个python package

* 内部需要有一个空的初始化文件

```python
__init__.py
```



### 02, 然后配置配置文件db_config.ini

```python
[DATABASE]
host=127.0.0.1
port=3306
user=root
passwd=,.
database=test
dbchar=utf8
```



### 03, 编写配置文件读取IMConfig.py

```python
# author      : ivanl001
# creator     : 2019/10/19 15:45
# description : 读取配置信息工具类

import os
import configparser

# 得到readConfig.py文件的上级目录
path = os.path.split(os.path.realpath(__file__))[0]

# 得到配置文件目录，配置文件目录为path下的\config.ini
config_path = os.path.join(path, 'db_config.ini')
# 调用配置文件读取
config = configparser.ConfigParser()
config.read(config_path, encoding='utf-8')


# 定义一个类，方便后续进行调用相关配置，静态方法哈
def get_mysql_attr(name):
    value = config.get('DATABASE', name)
    return value


if __name__ == '__main__':
    # # 测试path内容
    # print('path值为：', path)
    # # 打印输出config_path测试内容是否正确
    # print('config_path', config_path)
    # # 通过上面的ReadConfig().get_mysql方法获取配置文件中DATABASE的'host'的对应值为10.182.27.158
    # print('通过config.get拿到配置文件中DATABASE的host的对应值:', IMInnerConfig.get_mysql('host'), IMInnerConfig.get_mysql("port"))
    #
    # print("----")
    # print(IMInnerConfig.get_mysql("host"))
    print(get_mysql_attr("host"))

```



### 04, 编写数据库连接文件IMDBPool.py

```python
# author      : ivanl001
# creator     : 2019/10/19 15:54
# description : 数据库连接池工具类

# -*- coding: UTF-8 -*-
"""
1、执行带参数的ＳＱＬ时，请先用sql语句指定需要输入的条件列表，然后再用tuple/list进行条件批配
２、在格式ＳＱＬ中不需要使用引号指定数据类型，系统会根据输入参数自动识别
３、在输入的值中不需要使用转意函数，系统会自动处理
"""
import pymysql
from DBUtils.PooledDB import PooledDB

# from ivanl001 import IMConfig
from ivanl001.IMConfig import get_mysql_attr


"""
Config是一些数据库的配置文件,通过调用我们写的readConfig来获取配置文件中对应值
"""
host = get_mysql_attr('host')
port = int(get_mysql_attr('port'))
user = get_mysql_attr('user')
passwd = get_mysql_attr('passwd')
database = get_mysql_attr('database')
dbchar = get_mysql_attr('dbchar')


class Mysql(object):
    """
    MYSQL数据库对象，负责产生数据库连接 , 此类中的连接采用连接池实现获取连接对象：conn = Mysql.getConn()
            释放连接对象;conn.close()或del conn
    """
    # 连接池对象
    __pool = None

    def __init__(self):
        # 数据库构造函数，从连接池中取出连接，并生成操作游标
        self._conn = Mysql.__getConn()
        self._cursor = self._conn.cursor()

    @staticmethod
    def __getConn():
        """
        @summary: 静态方法，从连接池中取出连接
        @return MySQLdb.connection
        """
        if Mysql.__pool is None:
            __pool = PooledDB(creator=pymysql, mincached=1, maxcached=20, host=host, port=port, user=user,
                              passwd=passwd, db=database)
        return __pool.connection()

    def getAll(self, sql, param=None):
        """
        @summary: 执行查询，并取出所有结果集
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list(字典对象)/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchall()
        else:
            result = False
        return result

    def getOne(self, sql, param=None):
        """
        @summary: 执行查询，并取出第一条
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchone()
        else:
            result = False
        return result

    def getMany(self, sql, num, param=None):
        """
        @summary: 执行查询，并取出num条结果
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param num:取得的结果条数
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchmany(num)
        else:
            result = False
        return result

    def insertOne(self, sql, value):
        """
        @summary: 向数据表插入一条记录
        @param sql:要插入的ＳＱＬ格式
        @param value:要插入的记录数据tuple/list
        @return: insertId 受影响的行数
        """
        self._cursor.execute(sql, value)
        return self.__getInsertId()

    def insertMany(self, sql, values):
        """
        @summary: 向数据表插入多条记录
        @param sql:要插入的ＳＱＬ格式
        @param values:要插入的记录数据tuple(tuple)/list[list]
        @return: count 受影响的行数
        """
        count = self._cursor.executemany(sql, values)
        return count

    def __getInsertId(self):
        """
        获取当前连接最后一次插入操作生成的id,如果没有则为０
        """
        self._cursor.execute("SELECT @@IDENTITY AS id")
        result = self._cursor.fetchall()
        return result[0]['id']

    def __query(self, sql, param=None):
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        return count

    def update(self, sql, param=None):
        """
        @summary: 更新数据表记录
        @param sql: ＳＱＬ格式及条件，使用(%s,%s)
        @param param: 要更新的  值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def delete(self, sql, param=None):
        """
        @summary: 删除数据表记录
        @param sql: ＳＱＬ格式及条件，使用(%s,%s)
        @param param: 要删除的条件 值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def begin(self):
        """
        @summary: 开启事务
        """
        self._conn.autocommit(0)

    def end(self, option='commit'):
        """
        @summary: 结束事务
        """
        if option == 'commit':
            self._conn.commit()
        else:
            self._conn.rollback()

    def dispose(self, isEnd=1):
        """
        @summary: 释放连接池资源
        """
        if isEnd == 1:
            self.end('commit')
        else:
            self.end('rollback')
        self._cursor.close()
        self._conn.close()


if __name__ == '__main__':
    print(host, port, user, passwd, database)
```



### 05, 测试使用abc_sql.py

* 这个类不能test开头，目前还不太懂python的测试

```python
# # author      : ivanl001
# # creator     : 2019/10/19 16:53
# # description : 注意：文件名不要以test开头
#
# coding:utf-8
from ivanl001 import IMDBPool

mysql = IMDBPool.Mysql()

# sql语句，具体根据实际情况填写真实信息
sqlAll = "select * from person"
result = mysql.getAll(sqlAll, None)
if result:
    print("get all")
    for row in result:
        id01 = row[0]
        name = row[1]
        age = row[2]
        # 这里要做类型转换，和java不一样，不同类型不能一起打印
        print(str(id01) + ":" + name + ":" + str(age))

# 释放连接池资源
mysql.dispose()
```



## 3, python面向对象编程

```python
#
# author      : ivanl001
# creator     : 2018-12-06 16:54
# description : 该文件描述对象构造， 对象方法，静态方法等
# 

# 定义类
class Dog:

    # 01, 定义构造函数, 如果有了下面的带参数的构造器，这里的构造器就不能再写了
    def __init__(self):
        print("A:定义构造函数")

    def __init__(self, name, age, num):
        #下面这三个可以直接理解成定义成员变量
        self.name = name
        self.age = age
        self.num = num
        print("A:定义构造函数")

    def __del__(self):
        print("这个是对象销毁方法，也即是析构函数")

    # 02, 定义属性变量, 这里和静态成员变量还是有一定差别的， 这里的可以被更改的哈
    name = "ivanl001"

    # 03, 定义方法
    @staticmethod
    def play():
        print("C:定义并调用方法: dog is playing")

    @staticmethod
    def add(a:int, b:int):
        return a+b

    def print_name(self):
        print("打印成员变量：" + self.name)


# 创建对象(python和java的一个不同就是python可以直接在同一个文件中定义类后并创建
# dog01 = Dog()

# 因为后来的构造方法新增了变量，所以在构造dog的时候需要添加参数
dog = Dog("ivanl002", 10, 10120945)

print("B:打印静态成员变量name:" + dog.name)

# play是静态方法，可以直接对象调用，也可以类调用哈
dog.play()
Dog.play()

# 对象方法
dog.print_name()
# 注意：这里不是静态方法，所以不能直接用Dog类进行调用
# Dog.printName()


print("---------------对象属性-----------------")
# 判断对象是否有某个变量， 也可以判断是否有某个方法的
print(hasattr(dog, "name"))
print(hasattr(dog, "add"))
# 获取属性值
print(getattr(dog, "name"))
# 更改属性值
setattr(dog, "name", "ivanl003")
print(getattr(dog, "name"))
# 删除属性值
delattr(dog, "age")
print(hasattr(dog, "age"))


# 类属性
print("----------------类属性-----------------")
dogDict = Dog.__dict__
for key in dogDict.keys():
    print("key:" + key + ", value:" + str(dogDict[key]))
```



## 4, 多线程

```python 
#
# author      : ivanl001
# creator     : 2018-12-06 16:21
# description : 
# 

# 这个是高级接口，它的低级接口是_thread
import threading

# 这个是threading的低级接口
import _thread


# 定义一个函数，方便多线程演示
def hello():
    tname = threading.current_thread().getName()
    print(tname)
    print("ivanl001 is the king of world!")
    for i in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
        print(tname + ":" + str(i))


# 注意一下：这里就算没有参数，也需要传递进去一个空的元祖
threading._start_new_thread(hello,())
threading._start_new_thread(hello,())
#_thread.start_new_thread(hello, ())

# 这里一定要注意：主线程不能死，祝线程一旦死掉，分线程就不能打印出内容了
# 如果这样的话，比较容易太耗资源,所以还是和java一样，用休眠的方法
# while True:
#     pass

import time

# 获取当前毫秒值
current = int(time.time()*1000)
print(current)

# 利用time模块让主进程休眠2秒钟
time.sleep(2)

```



## 5, 爬虫相关

### 01, 用urllib3进行简单获取

```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:59
# description : 爬虫基础, 这里简单写下代码， 在spark的0810小节有更多的内容，这里就不再练习了，后面有需要再复习吧
# 

import urllib3

# 1，这里是创建连接池
http = urllib3.PoolManager()

# 2, 进行请求
# 这里可以是网页
# result = http.request("GET", "https://www.python.org")
# 也可以是图片
result = http.request("GET", "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1544264701928&di=2ce9bf1e5971e4057359e28ec0702613&imgtype=0&src=http%3A%2F%2Fgss0.baidu.com%2F-vo3dSag_xI4khGko9WTAnF6hhy%2Fzhidao%2Fpic%2Fitem%2F960a304e251f95cafc0b22edcf177f3e660952ea.jpg")

# 3，打印结果
print(result.status)
# print(result.data)

# 4，保存到文件
# output01 = open("/Users/ivanl001/Desktop/zhang.html","wb")
output01 = open("./zhang.jpg","wb")
output01.write(result.data)
```

### 02, 用urllib3进行迭代获取网站数据

```python
#
# author      : ivanl001
# creator     : 2018-12-08 13:59
# description : 爬虫基础, 这里简单写下代码， 在spark的0810小节有更多的内容，这里基本实现了，但是还是有点小问题，如果网页没有回应，就会一直等待，这个明显是不行的，但是暂时还没有找到合适的方案
# 

import urllib3
import re
import time
import os


# 1，这里是创建连接池
http = urllib3.PoolManager()


# 获取一个页面到url
def get_url_from_page(page_str):
    # 1, 解析
    pattern = u"<a\\s*href=\"([\u0000-\uffff]*?)\".*?>"
    # pattern = u'<a[\u0000-\uffff&&^[href]]*href="([\u0000-\uffff&&^"]*?)"'
    urls = re.finditer(pattern, page_str)
    for url in urls:
        addr = url.group(1)
        if str(addr).startswith("http"): # 这里做的比较简单，只判断http开头的，其他的一概不要了
            print(addr)
            # 这里要先判断一下这个文件是否已经存在，如果已经存在就不能再下载了
            name = addr.replace(":", "").replace("//","").replace("/","")
            outputPath = "/Users/ivanl001/Desktop/zhang/" + name + ".html"
            if os.path.exists(outputPath):
                continue
            else:
                # 这里就是相当于循环，一直在不停的循环迭代进行获取网页
                download_page(addr)
        else:
            continue


# 保存到文件
def download_page(urlStr):

    # 1, 进行请求, 这里应该设置一个超时时间什么的， 因为现在有一个问题，如果其中一个网页无响应，就会卡在那里一直等，这个后续
    result = http.request("GET", urlStr)

    # 2，获取(打印)结果
    resultByte = result.data
    # print(result.data)

    # 3, 对获取结果进行解码，并进行保存到文件，这里命名按照时间先来吧
    # 这里命名还是不能用时间，要不没法判断是否已经存在
    # current_time = int(time.time()*100000)
    name = urlStr.replace(":", "").replace("//","").replace("/","")
    outputPath = "/Users/ivanl001/Desktop/zhang/" + name + ".html"
    print(outputPath)
    output = open(outputPath ,"wb")
    output.write(resultByte)

    # 5, 对获取页面进行解码，并获取其中对url
    pageStr = resultByte.decode("utf-8")
    # print(pageStr)
    get_url_from_page(pageStr)


url = "http://focus.tianya.cn/"
# download_page("http://focus.tianya.cn/")
download_page(url)
```



