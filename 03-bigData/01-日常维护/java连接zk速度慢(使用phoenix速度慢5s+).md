使用phoenix速度慢，是因为phoenix每次都需要从zk中拿数据。

刚开始发现，每次查询都需要5s+，以为是phoenix设置了限制，但是找来找去没有找到相关内容

然后开始想，是不是zerotier有限制？也没找到相关内容。

再之后， 就是想到phoenix要用到zk，是不是zk的问题？然后连接了一下zk，发现也是5s+。

然后开始搜索zk速度慢的原因， 然后就找到了hostname

经过一番的查找，最终终于找到了原因：是因为mac的hostname设置的问题！！！！





所以连接phoenix速度慢的真实原因是因为连接zk慢。





而连接zk速度慢的原因是：/etc/hosts里面的hostname的配置！！！！

艹艹艹艹艹！



太恶心了， 纠结这么久的东西原来是因为host配置！！！！！



参考文档：

zk速度慢相关内容

https://www.pianshen.com/article/17161504500/

主机名相关内容

https://blog.csdn.net/u010953692/article/details/82845619

```shell
scutil --get HostName
scutil --get LocalHostName
scutil --get ComputerName
```

