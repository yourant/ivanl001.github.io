[toc]

参考文档：https://zhuanlan.zhihu.com/p/137055447



> Flink Runtime 层的整个架构采用了标准 **Master-Slave** 的结构
>
> Flink运行时架构主要包括四个不同的组件：
>
> 资源管理器（ResourceManager）、
>
> 作业管理器（JobManager）、
>
> 任务管理器（TaskManager），
>
> 以及分发器（Dispatcher）



## 1, 运行时架构

![img](https://pic2.zhimg.com/80/v2-d94d8ca8597a60aff269dcae21db39c9_1440w.jpg)



### 1）Dispatcher

* 负责接收用户提供的作业，并且负责为这个新提交的作业拉起一个新的 JobManager 组件。



### 2)  资源管理器（Resourc eManager）

* 负责资源的管理，在整个 Flink 集群中只有一个 ResourceManager



### 3）JobManager

* 相当于spark的Driver
* 负责管理作业的执行，在一个 Flink 集群中可能有多个作业同时执行，每个作业都有自己的 JobManager 组件



### 4）TaskManager

* 相当于spark的Exector
* TaskManager是一个Flink集群的工作（worker）进程。
* 任务（Tasks）被调度给TaskManager执行。它们彼此通信以在后续任务之间交换数据





## 2, 执行图

Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图

* StreamGraph：
  * 是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构

* JobGraph：
  * StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点**chain在一起**作为一个节点

* ExecutionGraph：
  * JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。

* 物理执行图：
  * JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。



