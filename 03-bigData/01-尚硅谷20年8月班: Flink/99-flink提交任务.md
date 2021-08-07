flink提交任务按照作业模式提交

也就是每次提交都会创建一个新的flink集群，为每个job提供一个yarn-session，任务之间相互独立，互不影响，方便管理。任务执行完毕之后，集群也会关闭

脚本如下：

```shell
bin/yarn-session.sh 	-n 7 -s 8 -jm 3072 -tm 32768 -qu root.*.*-nm *-* -d 
# 该任务申请7个taskmananger，每个taskmanager是8核， 32768M内存(tm:task manager)
# jm:job manager, 作业管理器是3072M
```





