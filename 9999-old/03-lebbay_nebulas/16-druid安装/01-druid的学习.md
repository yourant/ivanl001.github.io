[toc]



参考：

[https://www.wangt.cc/2018/08/%e3%80%90%e5%b9%b2%e8%b4%a7%e3%80%91%e4%b8%80%e6%96%87%e7%90%86%e8%a7%a3druid%e5%8e%9f%e7%90%86%e6%9e%b6%e6%9e%84%ef%bc%88%e6%97%b6%e5%ba%8f%e6%95%b0%e6%8d%ae%e5%ba%93%ef%bc%8c%e4%b8%8d%e6%98%afali/](https://www.wangt.cc/2018/08/[干货]一文理解druid原理架构（时序数据库，不是ali/)





### Druid本身包含5个组成部分：

Broker nodes, Historical nodes, Realtime nodes, Coordinator Nodes和indexing services. 分别的作用如下：

- Broker nodes: 负责响应外部的查询请求，通过查询Zookeeper将请求划分成segments分别转发给Historical和Real-time nodes，最终合并并返回查询结果给外部；
- Historial nodes: 负责’Historical’ segments的存储和查询。其会从deep storage中load segments，并响应Broder nodes的请求。Historical nodes通常会在本机同步deep storage上的部分segments，所以即使deep storage不可访问了，Historical nodes还是能serve其同步的segments的查询；
- Real-time nodes: 用于存储和查询热数据，会定期地将数据build成segments移到Historical nodes。一般会使用外部依赖kafka来提高realtime data ingestion的可用性。如果不需要实时ingest数据到cluter中，可以舍弃Real-time nodes，只定时地batch ingestion数据到deep storage；
- Coordinator nodes: 可以认为是Druid中的master，其通过Zookeeper管理Historical和Real-time nodes，且通过Mysql中的metadata管理Segments
- Druid中通常还会起一些indexing services用于数据导入，batch data和streaming data都可以通过给indexing services发请求来导入数据。





#### **Broker node**

- Broker节点扮演着历史节点和实时节点的查询路由的角色。
- Broker节点知道发布于Zookeeper中的关于哪些segment是可查询的和这些segment是保存在哪里的，Broker节点就可以将到来的查询请求路由到正确的历史节点或者是实时节点，
- Broker节点也会将历史节点和实时节点的局部结果进行合并，然后返回最终的合并后的结果给调用者

**缓存：**Broker节点包含一个支持LRU失效策略的缓存。这个缓存可以使用本地堆内存或者是一个外部的分布式 key/value 存储，例如Memcached

- 每次Broker节点接收到查询请求时，都会先将查询映射到一组segment中去。这一组确定的segment的结果可能已经存在于缓存中，而不需要重新计算。
- 对于那些不存在于缓存的结果，Broker节点会将查询转发到正确的历史节点和实时节点中去，一旦历史节点返回结果，Broker节点会将这些结果缓存起来以供以后使用，这个过程如下图所示
- 注意：**实时数据永远不会被缓存**，因此查询实时节点的数据的查询请求总是会被转发到实时节点上去。实时数据是不断变化的，因此缓存实时数据是不可靠的
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/6c23facf5a3df23d570a73cdf5b012b3.png)
- 上图：结果会为每一个segment缓存。查询会合并缓存结果与历史节点和实时节点的计算结果
- 缓存也可作为数据可用性的附加级别。在所有历史节点都出现故障的情况下，对于那些命中已经在缓存中缓存了结果的查询，仍然是可以返回查询结果的

**可用性：**在所有的Zookeeper都中断的情况下，数据仍然是可以查询的。如果Broker节点不可以和Zookeeper进行通信了，它会使用它最后一次得到的整个集群的视图来继续将查询请求转发到历史节点和实时节点，Broker节点假定集群的结构和Zookeeper中断前是一致的。在实践中，在我们诊断Zookeeper的故障的时候，这种可用性模型使得Druid集群可以继续提供查询服务，为我们争取了更多的时间

 **说明：**通常在ShareNothing的架构中,如果一个节点变得不可用了,会有一个服务将下线的这个节点的数据搬迁到其他节点，但是如果这个节点下线后又立即重启,而如果服务在一下线的时候就开始搬迁数据,是会产生跨集群的数据传输,实际上是没有必要的。因为分布式文件系统对同一份数据会有多个副本,搬迁数据实际上是为了满足副本数.而下线又重启的节点上的数据不会有什么丢失的，因此短期的副本不足并不会影响整体的数据健康状况.何况跨机器搬迁数据也需要一定的时间,何不如给定一段时间如果它真的死了,才开始搬迁



#### **Historical node** 

- 历史节点封装了加载和处理由实时节点创建的不可变数据块（segment）的功能。在很多现实世界的工作流程中，大部分导入到Druid集群中的数据都是不可变的，因此，历史节点通常是Druid集群中的主要工作组件。
- 历史节点遵循shared-nothing的架构，因此节点间没有单点问题。节点间是相互独立的并且提供的服务也是简单的，它们只需要知道如何加载、删除和处理不可变的segment *(注：shared nothing architecture是一 种分布式计算架构，这种架构中不存在集中存储的状态，整个系统中没有资源竞争，这种架构具有非常强的扩张性，在web应用中广泛使用）*
- 类似于实时节点，历史节点在Zookeeper中通告它们的在线状态和为哪些数据提供服务。加载和删除segment的指令会通过Zookeeper来进行发布，指令会包含segment保存在deep storage的什么地方和怎么解压、处理这些segment的相关信息
- 在历史节点从deep storage下载某一segment之前，它会先检查本地缓存信息中看segment是否已经存在于节点中，如果segment还不存在缓存中，历史节点会从deep storage中下载segment到本地
- 一旦处理完成，这个segment就会在Zookeeper中进行通告。此时，这个segment就可以被查询了。历史节点的本地缓存也支持历史节点的快速更新和重启，在启动的时候，该节点会检查它的缓存，并为任何它找到的数据立刻进行服务的提供，如下图：
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/bb82463965dd59848d3efb5076565865.png)

- 历史节点从deep storage下载不可变的segment。segment在可以被查询之前必须要先加载到内存中
- 历史节点可以支持读一致性，因为它们只处理不可变的数据。不可变的数据块同时支持一个简单的并行模型：历史节点可以以非阻塞的方式并发地去扫描和聚合不可变的数据块

**Tiers:** 历史节点可以分组到不同的tier中，哪些节点会被分到一个tier中是可配置的。Tier的目的是可以根据segment的重要程度来分配高或低的优先级来进行数据的分布。

- 可以为不同的tier配置不同的性能和容错参数。例如，可以使用一批很多个核的CPU和大容量内存的节点来组成一个“热点数据”的tier，这个“热点数据”集群可以配置来用于下载更多经常被查询的数据。
- 一个类似的”冷数据”集群可以使用一些性能要差一些的硬件来创建，“冷数据”集群可以只包含一些不是经常访问的segment

**可用性:** 历史节点依赖于Zookeeper来管理segment的加载和卸载。

 

- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/ffd2f42aad81f197c3f84e98de68f3d7.png)
- 如果Zookeeper变得不可用的时候，历史节点就不再可以为新的数据提供服务和卸载过期的数据，因为是通过HTTP来为查询提供服务的
- 对于那些查询它当前已经在提供服务的数据，历史节点仍然可以进行响应。这意味着Zookeeper运行故障时不会影响那些已经存在于历史节点的数据的可用性。



#### Realtime node

- 实时节点封装了导入和查询事件数据的功能，经由这些节点导入的事件数据可以立刻被查询。
- 实时节点只关心一小段时间内的事件数据，并定期把这段时间内收集的这批不可变事件数据导入到Druid集群里面另外一个专门负责处理不可变的批量数据的节点中去。
- 实时节点通过Zookeeper的协调和Druid集群的其他节点协调工作。实时节点通过Zookeeper来宣布他们的在线状态和他们提供的数据
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/c89cc31fd3cf8cbe91398e7d83558d48.png)
- 实时节点为所有传入的事件数据维持一个内存中的索引缓存, 随着事件数据的传入，这些索引会逐步递增，并且这些索引是可以立即查询的，查询这些缓存于JVM的基于堆的缓存中的事件数据，Druid就表现得和行存储一样
- 为了避免堆溢出问题，实时节点会定期地、或者在达到设定的最大行限制的时候，把内存中的索引持久化到磁盘去
- 这个持久化进程会把保存于内存缓存中的数据转换为基于列存储的格式，所有持久化的索引都是不可变的，并且实时节点会加载这些索引到off-heap内存中使得它们可以继续被查询
- 上图实时节点缓存事件数据到内存中的索引上，然后有规律的持久化到磁盘上。在转移之前，持久化的索引会周期性地合并在一起。查询会同时命中内存中的和已持久化的索引
- 所有的实时节点都会周期性的启动后台的计划任务搜索本地的持久化索引，后台计划任务将这些持久化的索引合并到一起并生成一块不可变的数据，这些数据块包含了一段时间内的所有已经由实时节点导入的事件数据，我们称这些数据块为”Segment”。在传送阶段，实时节点将这些segment上传到一个永久持久化的备份存储中，通常是一个分布式文件系统，例如S3或者HDFS，Druid称之为”Deep Storage”。

**实时节点处理流程：**导入、持久化、合并和传送这些阶段都是流动的，并且在这些处理阶段中不会有任何数据的丢失，数据流图如下：

![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/19cf133049c44b18860cc2c619c3c9b8.png)

 

- 节点启动于13:47，并且只会接受当前小时和下一小时的事件数据。当事件数据开始导入后，节点会宣布它为13:00到14:00这个时间段的Segment数据提供服务
- 每10分钟（这个时间间隔是可配置的），节点会将内存中的缓存数据刷到磁盘中进行持久化，在当前小时快结束的时候，节点会准备接收14:00到15:00的事件数据，一旦这个情况发生了，节点会准备好为下一个小时提供服务，并且会建立一个新的内存中的索引。
- 随后，节点宣布它也为14:00到15:00这个时段提供一个segment服务。节点并不是马上就合并13:00到14:00这个时段的持久化索引，而是会等待一个可配置的窗口时间，直到所有的13:00到14:00这个时间段的一些延迟数据的到来。这个窗口期的时间将事件数据因延迟而导致的数据丢失减低到最小。
- 在窗口期结束时，节点会合并13:00到14:00这个时段的所有持久化的索引合并到一个独立的不可变的segment中，并将这个segment传送走，一旦这个segment在Druid集群中的其他地方加载了并可以查询了，实时节点会刷新它收集的13:00到14:00这个时段的数据的信息，并且宣布取消为这些数据提供服务。



#### **Coordinator node**

- 主要负责数据的管理和在历史节点上的分布。协调节点告诉历史节点加载新数据、卸载过期数据、复制数据、和为了负载均衡移动数据。
- Druid为了维持稳定的视图，使用一个多版本的并发控制交换协议来管理不可变的segment。如果任何不可变的segment包含的数据已经被新的segment完全淘汰了，则过期的segment会从集群中卸载掉。
- 协调节点会经历一个leader选举的过程，来决定由一个独立的节点来执行协调功能，其余的协调节点则作为冗余备份节点
- 协调节点会周期性（一分钟）的执行来确定集群的当前状态，它通过在运行的时候对比集群的预期状态和集群的实际状态来做决定。和所有的Druid节点一样，协调节点维持一个和Zookeeper的连接来获取当前集群的信息（数据拓扑图、元信息库中所有有效的Segment信息以及规则库）
- 协调节点也维持一个与MySQL数据库的连接，MySQL包含有更多的操作参数和配置信息。
- 其中一个存在于MySQL的关键信息就是历史节点可以提供服务的所有segment的一个清单，这个表可以由任何可以创建segment的服务进行更新，例如实时节点。
- MySQL数据库中还包含一个Rule表来控制集群中segment的是如何创建、销毁和复制

##### **Rules：**Rules管理历史segment是如何在集群中加载和卸载的。

- Rules指示segment应该如何分配到不同的历史节点tier中，每一个tier中应该保存多少份segment的副本。
- Rules还可能指示segment何时应该从集群中完全地卸载。Rules通常设定为一段时间，例如，一个用户可能使用Rules来将最近一个月的有价值的segment载入到一个“热点数据”的集群中，最近一年的有价值的数据载入到一个“冷数据”的集群中，而将更早时间前的数据都卸载掉。
- 协调节点从MySQL数据库中的rule表加载一组rules。Rules可能被指定到一个特定的数据源，或者配置一组默认的rules。协调节点会循环所有可用segment并会匹配第一条适用于它的rule

**负载均衡：**在典型的生产环境中，查询通常命中数十甚至上百个segment，由于每个历史节点的资源是有限的，segment必须被分布到整个集群中，以确保集群的负载不会过于不平衡。

- 要确定最佳的负载分布，需要对查询模式和速度有一定的了解。通常，查询会覆盖一个独立数据源中最近的一段邻近时间的一批segment。平均来说，查询更小的segment则更快
- 这些查询模式提出以更高的比率对历史segment进行复制，把大的segment以时间相近的形式分散到多个不同的历史节点中，并且使存在于不同数据源的segment集中在一起
- 为了使集群中segment达到最佳的分布和均衡，根据segment的数据源、新旧程度、和大小，开发了一个基于成本的优化程序

##### **副本/复制（Replication）：**

- 协调节点可能会告诉不同的历史节点加载同一个segment的副本。每一个历史节点tier中副本的数量是完全可配置。
- 设置一个高级别容错性的集群可以设置一个比较高数量的副本数。segment的副本被视为和原始segment一样的，并使用相同的负载均衡算法
- 通过复制segment，单一历史节点故障对于整个Druid集群来说是透明的，不会有任何影响

##### **可用性:**

- 协调节点有Zookeeper和MySQL这两个额外的依赖，协调节点依赖Zookeeper来确定集群中有哪些历史节点
- 如果Zookeeper变为不可用，协调节点将不可以再进行segment的分配、均衡和卸载指令的发送。不过，这些都不会影响数据的可用性
- 对于MySQL和Zookeeper响应失效的设计原则是一致的：如果协调节点一个额外的依赖响应失败了，集群会维持现状
- Druid使用MySQL来存储操作管理信息和关于segment如何存在于集群中的segment元数据。如果MySQL下线了，这些信息就在协调节点中变得不可用，不过这不代表数据不可用
- 如果协调节点不可以和MySQL进行通信，他们会停止分配新的segment和卸载过期的segment。在MySQL故障期间Broker节点、历史节点、实时节点都是仍然可以查询的



#### **Indexing server**

Indexing Service是负责“生产”Segment的高可用、分布式、Master/Slave架构服务。主要由三类组件构成：

负责运行索引任务(indexing task)的Peon，

负责控制Peon的MiddleManager，

负责任务分发给MiddleManager的Overlord；

三者的关系可以解释为：

Overlord是MiddleManager的Master，

而MiddleManager又是Peon的Master。

其中，Overlord和MiddleManager可以分布式部署，但是Peon和MiddleManager默认在同一台机器上



Overlord
Overlord负责接受任务、协调任务的分配、创建任务锁以及收集、返回任务运行状态给调用者。当集群中有多个Overlord时，则通过选举算法产生Leader，其他Follower作为备份。



Overlord可以运行在local（默认）和remote两种模式下，如果运行在local模式下，则Overlord也负责Peon的创建与运行工作，当运行在remote模式下时，Overlord和MiddleManager各司其职，根据图3.6所示，Overlord接受实时/批量数据流产生的索引任务，将任务信息注册到Zookeeper的/task目录下所有在线的MiddleManager对应的目录中，由MiddleManager去感知产生的新任务，同时每个索引任务的状态又会由Peon定期同步到Zookeeper中/Status目录，供Overlord感知当前所有索引任务的运行状况。



Overlord对外提供可视化界面，通过访问http://:/console.html，我们可以观察到集群内目前正在运行的所有索引任务、可用的Peon以及近期Peon完成的所有成功或者失败的索引任务。

MiddleManager
MiddleManager负责接收Overlord分配的索引任务，同时创建新的进程用于启动Peon来执行索引任务，每一个MiddleManager可以运行多个Peon实例。

在运行MiddleManager实例的机器上，我们可以在${ [java.io](http://java.io/).tmpdir}目录下观察到以XXX_index_XXX开头的目录，每一个目录都对应一个Peon实例；同时restore.json文件中保存着当前所有运行着的索引任务信息，一方面用于记录任务状态，另一方面如果MiddleManager崩溃，可以利用该文件重启索引任务。

Peon
Peon是Indexing Service的最小工作单元，也是索引任务的具体执行者，所有当前正在运行的Peon任务都可以通过Overlord提供的web可视化界面进行访问。

![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/41cdb99375d3f767036af56833bbfdea.png)

#### 架构介绍:

![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/5f773a2d482a6fe034ad476eed136469.png)

- 查询路径：红色箭头:①客户端向Broker发起请求,Broker会将请求路由到②实时节点和③历史节点
- Druid数据流转:黑色箭头：数据源包括实时流和批量数据. ④实时流经过索引直接写到实时节点，⑤批量数据通过IndexService存储到DeepStorage,⑥再由历史节点加载. ⑦实时节点也可以将数据转存到DeepStorage

- Druid的集群依赖了ZooKeeper来维护数据拓扑. 每个组件都会与ZooKeeper交互，如下：
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/35e5ee8c4a44abae43aaa3a2676501cf.png)
- 实时节点在转存Segment到DeepStorage, 会写入自己转存了什么Segment
- 协调节点管理历史节点,它负责从ZooKeeper中获取要同步/下载的Segment,并指派任务给具体的历史节点去完成
- 历史节点从ZooKeeper中领取任务,任务完成后要将ZooKeeper条目删除表示完成了任务
- Broker节点根据ZooKeeper中的Segment所在的节点, 将查询请求路由到指定的节点
- 对于一个查询路由路径,Broker只会将请求分发到实时节点和历史节点, 因此元数据存储和DeepStorage都不会参与查询中(看做是后台的进程).

#### **MetaData Storage 与 Zookeeper**

- MetaStore和ZooKeeper中保存的信息是不一样的. ZooKeeper中保存的是Segment属于哪些节点. 而MetaStore则是保存Segment的元数据信息
- 为了使得一个Segment存在于集群中,MetaStore存储的记录是关于Segment的自描述元数据: Segment的元数据,大小,所在的DeepStorage
- 元数据存储的数据会被协调节点用来知道集群中可用的数据应该有哪些(Segment可以通过实时节点转存或者批量数据直接写入).

 



### **Druid还包含3个外部依赖,与其说是依赖,不如说正式Druid开放的架构,用户可以根据自己的需求使用不同的外部组建**

 

- Mysql：存储Druid中的各种metadata（里面的数据都是Druid自身创建和插入的），在Druid_0.9.1.1版本中，元信息库[druid](https://www.wangt.cc/tag/druid/)主要包含十张表，均以“[druid](https://www.wangt.cc/tag/druid/)_”开头,例如张表：”[druid](https://www.wangt.cc/tag/druid/)_config”（通常是空的）, “[druid](https://www.wangt.cc/tag/druid/)_rules”（coordinator nodes使用的一些规则信息，比如哪个segment从哪个node去load）和“[druid](https://www.wangt.cc/tag/druid/)_segments”（存储每个segment的metadata信息）；
- Deep storage: 存储segments，Druid目前已经支持本地磁盘，NFS挂载磁盘，HDFS，S3等。Deep Storage的数据有2个来源，一个是[batch Ingestion](http://druid.io/docs/0.12.1/ingestion/batch-ingestion.html), 另一个是real-time nodes；
- ZooKeeper: Druid使用Zookeeper作为分布式集群内部的通信组件，各类节点通过Curator Framework将实例与服务注册到Zookeeper上，同时将集群内需要共享的信息也存储在Zookeeper目录下，从而简化集群内部自动连接管理、leader选举、分布式锁、path缓存以及分布式队列等复杂逻辑。

![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/image2018-8-616-14-19.png)

- ① 实时数据写入到实时节点,会创建索引结构的Segment.
- ② 实时节点的Segment经过一段时间会转存到DeepStorage
- ③ 元数据写入MySQL; 实时节点转存的Segment会在ZooKeeper中新增一条记录
- ④ 协调节点从MySQL获取元数据,比如schema信息(维度列和指标列)
- ⑤ 协调节点监测ZK中有新分配/要删除的Segment,写入ZooKeeper信息:历史节点需要加载/删除Segment
- ⑥ 历史节点监测ZK, 从ZooKeeper中得到要执行任务的Segment
- ⑦ 历史节点从DeepStorage下载Segment并加载到内存/或者将已经保存的Segment删除掉
- ⑧ 历史节点的Segment可以用于Broker的查询路由

 

- 由于各个节点和其他节点都是最小化解耦的, 所以下面两张图分别表示实时节点和批量数据的流程:
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/812b25aac73ac7f4dc208a61e18dbf2e.png)
- 数据从Kafka导入到实时节点, 客户端直接查询实时节点的数据.
- ![【干货】一文理解Druid原理架构（时序数据库，不是ali的数据库连接池）](https://www.wangt.cc/wp-content/uploads/2018/08/64e782717757d18536a8e7caa220a8ee.png)
- 批量数据使用IndexService,接收Post请求的任务,直接产生Segment写到DeepStorage里.DeepStorage中的数据只会被历史节点使用.
  所以这里要启动的服务有: IndexService(overlord), Historical, Coordinator(协调节点通知历史节点下载Segment),



 

 