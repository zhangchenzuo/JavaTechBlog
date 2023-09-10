- [spark 框架和运行原理](#spark-框架和运行原理)
  - [Spark组件](#spark组件)
  - [运行框架，基本架构](#运行框架基本架构)
  - [Driver上的执行环境--SparkContext](#driver上的执行环境--sparkcontext)
  - [Executor的执行环境--SparkEnv](#executor的执行环境--sparkenv)
  - [几种不同的部署方法](#几种不同的部署方法)
  - [集群运行（描述spark里一个任务的执行过程）](#集群运行描述spark里一个任务的执行过程)
  - [关键特性](#关键特性)
    - [数据本地性](#数据本地性)
    - [懒计算](#懒计算)
    - [容错](#容错)
- [调度系统](#调度系统)
- [spark的内存模型](#spark的内存模型)
  - [堆内内存](#堆内内存)
  - [堆外内存](#堆外内存)
  - [动态内存分配](#动态内存分配)
  - [可能的OOM](#可能的oom)
- [Job，Stage，Task](#jobstagetask)
  - [DAG/partition/Job](#dagpartitionjob)
  - [stage](#stage)
    - [依赖划分原则](#依赖划分原则)
    - [宽窄依赖](#宽窄依赖)
  - [Task](#task)
- [spark的数据结构](#spark的数据结构)
  - [RDD](#rdd)
    - [RDD的迭代计算](#rdd的迭代计算)
    - [容错机制](#容错机制)
    - [RDD持久化](#rdd持久化)
      - [persist/cache](#persistcache)
      - [如何选择一种最合适的持久化策略](#如何选择一种最合适的持久化策略)
  - [DataFrame](#dataframe)
  - [DataSet](#dataset)
  - [三种数据结构的对比](#三种数据结构的对比)
  - [分布共享变量](#分布共享变量)
    - [累加器 只写](#累加器-只写)
    - [广播变量 只读](#广播变量-只读)
  - [Checkpoint](#checkpoint)
    - [checkpoint和persist的区别](#checkpoint和persist的区别)
- [Shuffle](#shuffle)
  - [shuffle write](#shuffle-write)
    - [ExternalSorter](#externalsorter)
  - [shuffle read](#shuffle-read)
  - [shuffle优化](#shuffle优化)
  - [partitioner](#partitioner)
  - [聚合类算子](#聚合类算子)
- [Spark的Join实现](#spark的join实现)
  - [如何选择join机制](#如何选择join机制)
  - [shuffle hash join/ sort merge Join](#shuffle-hash-join-sort-merge-join)
- [Spark算子类型](#spark算子类型)
  - [transform 算子](#transform-算子)
  - [action算子](#action算子)
  - [算子例子](#算子例子)
- [与MR对比](#与mr对比)
  - [优点](#优点)
- [Spark调优](#spark调优)
  - [开发调优——（适当持久化，shuffle类算子使用优化，broadcast，更好的算子使用）](#开发调优适当持久化shuffle类算子使用优化broadcast更好的算子使用)
  - [资源调优](#资源调优)
  - [数据倾斜调优](#数据倾斜调优)
    - [现象和原理](#现象和原理)
    - [优化方案——数据倾斜 / join调优](#优化方案数据倾斜--join调优)
  - [shuffle调优](#shuffle调优)
    - [HashShuffleManager](#hashshufflemanager)
      - [shuffleWrite/shuffleRead](#shufflewriteshuffleread)
  - [sortShuffleManager](#sortshufflemanager)
- [spark 3.0](#spark-30)
- [Spark sql](#spark-sql)
  - [执行流程](#执行流程)
- [其他问题](#其他问题)
  - [spark的小文件问题](#spark的小文件问题)
    - [coalease和repartition的区别](#coalease和repartition的区别)
  - [Spark程序执行，有时候默认为什么会产生很多task，怎么修改默认task执行个数？](#spark程序执行有时候默认为什么会产生很多task怎么修改默认task执行个数)
  - [为什么shuffle性能差/shuffle发生了什么](#为什么shuffle性能差shuffle发生了什么)
  - [reduceByKey和groupByKey的区别](#reducebykey和groupbykey的区别)
  - [spark的序列化问题](#spark的序列化问题)
  - [描述一下spark的运行过程和原理](#描述一下spark的运行过程和原理)
  - [Executor的内存/core使用是什么样的](#executor的内存core使用是什么样的)
  - [使用spark实现topN](#使用spark实现topn)

# spark 框架和运行原理

## Spark组件
- Spark Core： 
将 分布式数据 抽象为弹性分布式数据集（RDD），实现了应用**任务调度**、RPC、序列化和压缩，并为运行在其上的上层组件提供API。它提供了**内存计算的能力**。
  - 基础设施
    - SparkConf：用于管理Spark应用程序的配置
    - RPC框架：早期用的Akka，后期改成了Netty实现。
    - ListenerBus：监听器模式异步调用的实现
    - 度量系统：监控各个组件的运行。
  - SparkContext：driver的运行环境
  - SparkEnv：spark的运行时环节。提供Executor运行环境。
  - 存储体系：BlockManager
  - 调度系统：
    - DAGScheduler：负责创建Job；将DAG中的RDD划分到不同的Stage；给Stage创建对应的Task；批量提奖Task。
    - TaskScheduler：按照FIFO或者FAIR等调度算法对批量Task进行调度；为task分配资源；将Task发送到Executor上执行。
  - 计算引擎：shuffleManager
  - ExecutorAllocationManager：基于工作负载动态分配和删除executor。定时调度，计算需要的executor数量。

- Spark SQL：
Spark Sql 是Spark来**操作结构化数据的程序包**，可以使用SQL语句的方式来查询数据。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。

## 运行框架，基本架构

master/worker指的是资源的调度层面，申请内存等。driver/executor则是在任务的执行调度层面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b90bf1d96c54e3099ffe06093cded08.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d5f1c3b621c4e3a80af25c4e771f01f.png)


- Master/Cluster Manager：控制整个集群，监控worker。对各个Worker上内存，CPU等资源的分配给application，但是不负责对executor分配资源。
在standalone模式中即为Master主节点，控制整个集群，监控worker。在YARN模式中为资源管理器。

- Worker：
从节点，受Cluster Manager资源管理，负责控制计算节点，启动Executor（或者Driver）。集群中任何一个可以运行spark应用代码的节点。**Worker就是物理节点**，可以在上面**启动Executor/driver进程**。一个worker上的memory、cpu由多个executor共同分摊。

- Driver：运行Application的main函数并**创建SparkContext**，**准备Spark应用程序的运行环境**。
  
  在Spark中由SparkContext负责与Cluster Manager通信，**进行资源申请、任务的分配和监控**等；当Executor部分运行完毕后，Driver同时负责**将SparkContext关闭**。
    1. 作业解析：将用户程序转化为任务。程序从输入数据创建一些系列RDD，使用转化操作派生出新的RDD，最后使用行动操作收集或者存储结果RDD数据。实际上是隐式的创建了一个由操作组成的逻辑上的**有向无环图（Directed Acyclic Graph， DAG）**。当驱动器运行时，会转化为物理执行计划。
    2. 为执行器节点调度任务：执行器启动后会向驱动器注册自己，每个执行器节点都代表一个能够执行任务和存储RDD数据的进程。执行器会把任务基于数据所在位置分配给合适的执行器进程。任务执行时，执行器会缓存这些数据，驱动器会跟踪这些数据的缓存位置，并利用这些信息来调度以后的任务，以减少网络传输。


- Executor：
在每个Worker上为某应用启动的一个**JVM进程**，该进程**负责运行Task，并且负责将数据存在内存或者磁盘上**，每个任务都有各自独立的Executor。Executor是一个执行Task的容器。任务相互独立，即使其他的执行器节点有崩溃也不会影响自身。它的主要职责是：
  1. 负责执行组成spark应用的任务，并将结果返回给驱动器进程
  2. 通过自身的Block Manager，为用户进程中要求缓存RDD提供内存式存储，RDD直接缓存在执行器内存中。

  每个Application都有自己专属的Executor进程，并且该进程在Application运行期间一直驻留。**Executor进程以多线程的方式运行Task**。

## Driver上的执行环境--SparkContext
Spark执行环境。
![在这里插入图片描述](https://img-blog.csdnimg.cn/df2798227e8348acb7e7d846d6f66a16.png)

核心的几个：
- SparkEnv：其实是executor的执行环境。但是local模式下也需要。
- DAGScheduler: 提交Job，切分stage，发送给TaskScheduler
- TaskScheduler: 调度task
  
## Executor的执行环境--SparkEnv

![在这里插入图片描述](https://img-blog.csdnimg.cn/ddd6fd6a202c4aefb25a82d75754d801.png)
- RPC环境
- 序列化管理器 SerializerManager
- 广播管理器 BroadcastManager
- map任务输出追踪器 MapOutputTrackerManager
- 输出提交协调器 OutputCommitCoordinator
- BlockManager、BlockManagerMaster 等存储体系。
- MemoryManager
- ShuffleManager
## 几种不同的部署方法
>本地模式

Spark不一定非要跑在hadoop集群，可以在本地，起多个线程的方式来指定。方便调试，本地模式分三类
  - local：只启动一个executor
  - local[k]: 启动k个executor
  - local：启动跟cpu数目相同的 executor

>standalone模式

分布式部署集群，自带完整的服务，资源管理和任务监控是Spark自己监控，这个模式也是其他模式的基础

>Spark on yarn模式

分布式部署集群，资源和任务监控交给yarn管理
粗粒度资源分配方式，包含cluster和client运行模式
  - cluster 适合生产，driver运行在集群子节点，具有容错功能
  - client 适合调试，driver运行在客户端

具体流程：yarn-client模式下作业执行流程：

1. 客户端生成作业信息提交给ResourceManager(RM)
2. RM在本地NodeManager启动container并将Application Master(AM)分配给该NodeManager(NM)
3. NM接收到RM的分配，启动Application Master并初始化作业，此时这个NM就称为Driver
4. Application向RM申请资源，分配资源同时通知其他NodeManager启动相应的Executor
5. Executor向本地启动的Application Master注册汇报并完成相应的任务

yarn-cluster模式下作业执行流程：

1. 客户端生成作业信息提交给ResourceManager(RM)
2. RM在**某一个NodeManager(由Yarn决定)启动container并将Application Master(AM)分配给该NodeManager(NM)**
3. NM接收到RM的分配，启动Application Master并初始化作业，此时这个NM就称为Driver
4. Application向RM申请资源，分配资源同时通知其他NodeManager启动相应的Executor
5. Executor向**NM上的Application Master**注册汇报并完成相应的任务

>Spark On Mesos模式

## 集群运行（描述spark里一个任务的执行过程）
（分区的数量取决于partition数量的设定，每个分区的数据只会在一个task中计算。）

![在这里插入图片描述](https://img-blog.csdnimg.cn/bef07a0b0a0d4df0a65861d70566385e.png)

1. Driver进程启动，创建一个SparkContext进行资源的申请、任务的分配和监控。通过RPCEnv发送请求到Master节点上,进行Spark应用程序的注册。
2. SparkContext向资源管理器（Standalone，Mesos，Yarn）申请运行Executor资源。Master在接受到Spark应用程序的注册申请之后，会发送给Worker进行资源的调度和分配。Worker 在接受Master的请求之后，启动Executor来分配资源。Executor启动分配资源好后，通过RPCEnv向Driver进行反注册。
3. SparkContext根据RDD的依赖关系构建DAG，DAG提交给DAGScheduler创建Job并且解析成不同的Stage。DAGScheduler根据Stage内RDD的partition数量创建多个task组（TaskSet）并且批量提交给底层调度器TaskScheduler处理。 
4. Executor向SparkContext申请Task，TaskScheduler按照FIFO或者FAIR等调度算法对批量Task进行调度，为task分配资源并将Task发放给Executor运行。提供应用程序代码。
5. SparkContext会在RDD转换开始前用BlockManager和BroadcastManager将任务配置进行广播。
6. Task在Executor上运行把执行结果反馈给TaskScheduler，然后反馈给DAGScheduler，运行完毕后写入数据并释放所有资源。

## 关键特性
### 数据本地性
在RDD，task等都有preferredLocations，非常关注数据本地性。移动数据不如移动计算。数据本地性分为了5个级别。

RDD的数据本地性来源于file的partition的位置，task的perfer来源于RDD。 MapOutputTracker记录数据的位置，包括task和location，shuffle location。
对于shuffle，我们也希望去多拿partition，因此会把计算放到data数据最多的机器上。
### 懒计算
1. 只有RDD被提交job的时候，才会去进行计算
2. 在job完成以后不会存储任何东西，只把结果返回driver
3. RDD的计算是按照partition为粒度去递归调用的
### 容错
1. task如果失败了可以重试
2. 对于node和executor有黑名单机制，黑名单会有超时机制

# 调度系统
1. 用户提交的job会被转换成一系列的RDD并通过RDD的依赖关系构建DAG，然后将RDD组成的DAG提交给调度系统
2. DAGScheduler负责接收RDD组成的DAG，将一系列RDD划分成不同的stage。根据stage不同的类型，给每个stage中未完成的partition创建不同的类型的task。每个stage有多少未完成的partition，就有多少task。每个stage的task会以taskset的形式，提交给taskscheduler。
3. TaskScheduler负责对taskset进行管理，并将TaskSetManager添加到调度池，然后将task的调度交给调度后端接口（SchedulerBackend）。SchedulerBackend申请taskScheduler，按照task调度算法对调度池中的所有TaskSetManager进行排序，然后对TaskSet按照最大本地性原则分配资源，并且在各个节点执行task
4. execute task：执行任务。

- DAGScheduler: 提交Job，切分stage，发送给TaskScheduler
- TaskScheduler: 调度task，把task发送给executor执行。
# spark的内存模型
物理上分类可以分为堆内内存和堆外内存。逻辑上可以分为计算内存和存储内存。

spark除了把内存作为计算资源以外，还作为了存储资源。MemoryManager负责管理。

在Spark 1.5及之前版本中，内存管理默认实现是StaticMemoryManager，称为**静态内存管理**。从Spark 1.6.0版本开始，Spark默认采用一种新的内存管理模型UnifiedMemoryManager，称为统一内存管理，其特点是可以动态调整Execution和Storage的内存，因此又称为**动态内存管理**。


内存可以被分为主要分为两类：
- Execution Memory：主要用于计算，如shuffles、joins、sorts及aggregations等操作
- Storage Memory：主要用于cache数据和在集群内部传输数据。

也可以Spark内存划分为堆内存和堆外内存。堆内存是JVM堆内存的一部分，堆外内存是工作节点中系统内存的一部分空间。

## 堆内内存
- Execution Memory：主要用于shuffles、joins、sorts及aggregations等计算操作，又称为Shuffle Memory。
- Storage Memory：主要用于cache数据、unroll数据，有时也被称为Cache Memory。
- User Memory：用户内存，主要用于存储内部元数据、用户自定义的数据结构等，根据用户实际定义进行使用。
- Reserved Memory：默认300M的系统预留内存，主要用于程序运行。
  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c98eb9ce8394456c80b13607d651a229.png)
参数spark.memory.fraction在Spark 2.x版本中默认0.6，即Spark Memory（Execution Memory + Storage Memory）默认占整个usableMemory（systemMemory - Reserved Memory）内存的60%。

参数spark.memory.storageFraction默认0.5，代表Storage Memory占用Spark Memory百分比，默认值0.5表示Spark Memory中Execution Memory和Storage Memory各占一半。

## 堆外内存
为了进一步优化内存的使用，减小GC开销，Spark 1.6版本还增加了对Off-heap Memory的支持，Off-heap Memory默认是关闭的，开启须设置参数spark.memory.offHeap.enabled为true，并通过参数spark.memory.offHeap.size设置堆外内存大小，单位为字节。

堆外内存划分上没有了用户内存与预留内存，只包含Execution Memory和Storage Memory两块区域。
## 动态内存分配
意思是说，当Execution Memory有空闲，Storage Memory不足时，Storage Memory可以借用Execution Memory，反之亦然。Execution Memory可以让Storage Memory写到磁盘，收回被占用的空间。如果Storage Memory被Execution Memory借用，因为实现上的复杂度，却收回不了空间。


## 可能的OOM
- map执行内存溢出：单个map 中产生了大量的对象导致的。
解决方法：1. 增加堆内内存。2. 减少每个 Task 处理数据量，使每个 Task产生大量的对象时，Executor 的内存也能够装得下。具体做法可以在会产生大量对象的 map 操作之前调用 repartition 方法，分区成更小的块传入 map。
- shuffle后内存溢出：可能因为数据倾斜。或者读写开销太大了，可以预先map或者filter一下。
- driver 内存溢出：用户在 Dirver 端口生成大对象，比如创建了一个大的集合数据结构。解决方法：增加driver内存，检查是不是代码逻辑错误，在driver端做了操作。



# Job，Stage，Task
一个Application由一个Driver和若干个Job构成，一个Job由多个Stage构成，一个Stage由多个没有Shuffle关系的Task组成。
## DAG/partition/Job

> DAG图：
用来反映RDD之间的依赖关系。在创建RDD的时候会自然有这个抽象的概念。串联起来了所有的stage。

![在这里插入图片描述](https://img-blog.csdnimg.cn/12d9ed994680447eb0d020aaac7e4a76.png)

>partition

数据分区，一个RDD的数据可以被划分为多少个分区。Spark根据partition的数量来确定Task的数量。

分区太少，会导致较少的并发、数据倾斜、或不正确的资源利用。分区太多，导致任务调度花费比实际执行时间更多的时间。若没有配置分区数，默认的分区数是：所有执行程序节点上的内核总数。

一般一个core上由2-4个分区。
>Job

用户提交的作业。当RDD及其DAG被提交给DAGScheduler以后，DAGScheduler会将RDD的transform和action都视为一个job。

**一个Job包含多个RDD及作用于相应RDD上的各种操作，它包含很多task的并行计算**。它通常由一个或多个RDD转换操作和行动操作组成，这些操作会被划分为一些Stage。

## stage
是Job的基本调度单位，由**一组共享相同的Shuffle依赖关系的任务组成**。有一个job分割多个stage的的点在于shuffle，宽依赖Stage需要进行Shuffle操作，而窄依赖Stage则不需要。没有依赖关系的stage并行执行，有依赖关系的stage顺序执行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b0551be007e4a86a46ce7b8994e6fae.png)

比如某个job有一个reduceByKey，会被切分为两个stage。
  1. stage0: 从textFile到map，最后一步是**shuffle write**操作。我们可以简单理解为对pairs RDD中的数据进行分区操作，每个task处理的数据中，相同的key会写入同一个磁盘文件内。 
  2. stage1：主要是执行从reduceByKey到collect操作，stage1的各个task一开始运行，就会首先执行shuffle read操作。执行**shuffle read**操作的task，会从stage0的各个task所在节点**拉取属于自己处理的那些key**，然后对同一个key进行全局性的聚合或join等操作。

区分shuffleMapStage的原因是在做模式匹配的时候，对于shuffleMapStage可以把结果存储到MapOutputTracker上。也就是对于同一个Job的不同的stge，他们的结果是可见的。
### 依赖划分原则
NarrowDependency被分到同一个stage，这样可以管道的形式迭代执行，ShuffleDependency需要依赖多个分区。

容灾的角度：Narrow只需要重新执行父RDD的丢失分区的计算即可恢复，shuffle需要考虑恢复所有父RDD的丢失分区。

### 宽窄依赖
- 窄依赖：
指**父RDD的每一个分区最多被一个子RDD的分区所用**，表现为一个父RDD的分区对应于一个子RDD的分区，和两个父RDD的分区对应于一个子RDD 的分区。
子RDD依赖与父RDD中固定的Partiton，分为OneToOneDependency和RangeDependency


- 宽依赖：
**指子RDD的分区依赖于父RDD的所有分区，这是因为shuffle类操作**。子RDD对父RDD的所有分区都可能产生依赖。

> stage的分类
在Spark中，Stage可以分成两种类型。

- ShuffleMapStage：对shuffle数据映射到下游stage的各个分区
  - 这种Stage是以Shuffle为输出边界
  - 其输入边界可以是从外部获取数据，也可以是另一个ShuffleMapStage的输出
  - 其输出可以是另一个Stage的开始
  - ShuffleMapStage的最后Task就是ShuffleMapTask，通过shuffle连接下游数据
  - 在一个Job里可能有该类型的Stage，也可以能没有该类型Stage
- ResultStage
  - 这种Stage是直接输出结果
  - 其输入边界可以是从外部获取数据，也可以是另一个ShuffleMapStage的输出
  - ResultStage的最后Task就是ResultTask
  - 在一个Job里必定有该类型Stage
  
## Task
被发送到executor上的具体任务。每个Task负责计算一个分区的数据。

Task可以分为需要shuffle的**shuffleMapTask**和**resultTask**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6fc792ca49a459bb7d4eeae71364757.png)

用户提交的Job会提交给DAGScheduler，Job会被分解成Stage，Stage会被细化成Task，Task简单的说就是在一个数据partition上的单个数据处理流程。

当执行一个Application时，Driver会向集群管理器申请资源，启动Executor，并向Executor发送应用程序代码和文件，然后在Executor上执行Task，运行结束后，执行结果会返回给Driver，或者写到HDFS或者其它数据库中




需要注意的是，sprak都是**惰性计算**，只用在第一次的一个行动操作中用到时，才会真正计算。也就是说spark其实是了解了整个转化链后，只有真正执行才会计算。

# spark的数据结构
## RDD
弹性分布式数据集（Resilient Distributed Dataset， RDD）。是一个不可变（只读）的分布式对象集合，每个RDD可以有多个分区。它是spark的基础数据结构，具有内存计算能力、数据容错性以及数据不可修改特性。可以支持并行计算。

一个RDD包括一个或者多个分区，每个分区实际上是一个数据集合片段。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f03c7c353bf4245a720411a51dc3478.png)

> 为什么需要RDD：基本模型并行容错，依赖划分需要，并行执行需要，容错需要。

存放了三样最重要的数据：我的上游是什么，我是怎么计算的，我希望下游在哪里。

RDD的五个主要属性：
- partitions的列表。每个RDD是由多个分区组成的。
- 每个分区都有计算函数。compute函数
- 对其他RDD的依赖关系
- （可选）对kv RDD的partitioner，比如 Hash Partitioner 和 Range Partitioner。上游RDD通过partitioner知道输出给哪个下游RDD。
- （可选）计算每个分区的preferred location

包括的接口：
- getDependencies()
- getPartitions():Array[Partition]
- getPerferrededLocations(split: Partition): Seq[String]
- compute(split: Partition, context: TaskContext): Iterator[T]

模板方法：
- partitions: Array[Partitions]
- preferredLocations
- Map(): MapPartitionsRDD
- count(): Long


当调用RDD的partitons方法时，会先调用checkpoint的partitions，进而调用checkpoint的getPartitions方法，然后再调用RDD的getPartitions方法。
同理preferredLocations和compute。


>弹性

1. 内存的弹性：内存与磁盘的自动切换
2. 容错的弹性：数据丢失可以自动恢复
3. 计算的弹性：计算出错重试机制
4. 分片的弹性：根据需要重新分片 

### RDD的迭代计算
RDD的计算使用的时iterator方法作为入口，实际上在执行shuffleMapTask和ResultTask的runTask方法都会去调用RDD的iterator方法。

这个方法逻辑是：
1. 如果RDD的storageLevel不是None，尝试在磁盘，堆内内存，堆外内存去读取计算结果，调用getOrCompute方法。
2. 如果storageLevel是None，分区任务可能是首次执行，存储中还没有计算结果。所有调用computeOrReadCheckpoint方法计算或者从检查点恢复数据。

体现了容错思路，某个分区的task失败会导致stage的重新调度，此时成功的分区可以直接从checkpoint读取缓存的结果，错误的分区再次尝试计算。
### 容错机制
RDD的容错，主要是从保存了依赖关系上体现的。

　Spark框架层面的容错机制，主要分为三大层面（调度层、RDD血统层、Checkpoint层），在这三大层面中包括Spark RDD容错四大核心要点。

1. Stage输出失败，上层调度器DAGScheduler重试，重新调度stage。
2. Spark计算中，Task内部任务失败，底层调度器重试。
3. RDD Lineage血统中窄依赖、宽依赖计算。
4. Checkpoint缓存。 checkpoint通过将RDD写入Disk作检查点，是Spark lineage容错的辅助，lineage过长会造成容错成本过高，这时在中间阶段做检查点容错，如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做Lineage，就会减少开销。
### RDD持久化

在适当的位置persist/cache持久化RDD，可以防止重复计算。因为RDD是懒计算的，所以我们cache会分叉的RDD，可以防止这个RDD之前的流程再重复计算一遍。

让spark持久化存储一个RDD，计算出的RDD节点会分别保存它们所求出的分区数据。如果一个有持久化数据的节点发生故障，spark会在需要缓存数据时候重算。默认情况下，`persist()`会把数据以**序列化的形式缓存在JVM的堆空间**中。也可以通过改变参数，存在硬盘上。

持久化级别：https://tech.meituan.com/2016/04/29/spark-tuning-basic.html

#### persist/cache
cache和persist都是用于缓存RDD，避免重复计算.
.cache() == .persist(MEMORY_ONLY)

>cache后面能不能接其他算子,它是不是action操作？


可以接其他算子，但是接了算子之后，起不到缓存应有的效果，因为会重新触发cache。
cache不是action操作

#### 如何选择一种最合适的持久化策略
默认情况下，性能最高的当然是**MEMORY_ONLY**，但前提是你的内存必须足够足够大。因为不进行序列化与反序列化操作，就避免了这部分的性能开销；对这个RDD的后续算子操作，都是基于纯内存中的数据的操作，不需要从磁盘文件中读取数据，性能也很高；而且不需要复制一份数据副本，并远程传送到其他节点上。但是使用场景非常有限

如果使用MEMORY_ONLY级别时发生了内存溢出，那么建议尝试使用MEMORY_ONLY_SER级别。该级别会将RDD数据序列化后再保存在内存中，此时每个partition仅仅是一个字节数组而已，大大减少了对象数量，并降低了内存占用。这种级别比MEMORY_ONLY多出来的性能开销，主要就是序列化与反序列化的开销。但是后续算子可以基于纯内存进行操作，因此性能总体还是比较高的。

如果纯内存的级别都无法使用，那么建议使用MEMORY_AND_DISK_SER策略，而不是MEMORY_AND_DISK策略。因为既然到了这一步，就说明RDD的数据量很大，内存无法完全放下。序列化后的数据比较少，可以节省内存和磁盘的空间开销。同时该策略会优先尽量尝试将数据缓存在内存中，内存缓存不下才会写入磁盘。

通常不建议使用DISK_ONLY和后缀为_2的级别：因为完全基于磁盘文件进行数据的读写，会导致性能急剧降低，有时还不如重新计算一次所有RDD。后缀为_2的级别，必须将所有数据都复制一份副本，并发送到其他节点上，数据复制以及网络传输会导致较大的性能开销，除非是要求作业的高可用性，否则不建议使用。



## DataFrame
Dataframe也是一种不可修改的分布式数据集合，它可以按列查询数据，可以当成数据库里面的表看待。可以对**数据指定数据模式**（schema）。也就是说DataFrame是Dataset[row]的一种形式
## DataSet
Dataset是DataFrame的扩展，它提供了类型安全，面向对象的编程接口。知道每个row的字段类型。

更强的序列化能力，用了Tungsten。更加紧凑的存储了data
## 三种数据结构的对比
![在这里插入图片描述](https://img-blog.csdnimg.cn/41f79a3fb1a246428e68e9c78181fb90.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAemN6NTU2NjcxOQ==,size_18,color_FFFFFF,t_70,g_se,x_16)

- RDD类型，只是包含了对应的类型参数，但是并不清楚Person类的内部结构。而DataFrame可以得到Column的名字。
- RDD是分布式的 Java对象的集合。DataFrame是分布式的Row对象的集合。
- Dataset可以认为是DataFrame的一个特例，主要区别是Dataset每一个record存储的是一个强类型值而不是一个Row。

## 分布共享变量

两种类型的共享变量：累加器（accumulator）和广播变量（broadcast variable）；累加器用于对信息进行聚合，广播变量用于高效分发较大的变量。


### 累加器 只写
在向spark传递函数时候，可以使用驱动器程序中的变量，集群中运行的每个任务都会得到这些任务的一个新的副本，更新这些副本的值并不会影响驱动器中的值。

而累加器是一个共享变量，将工作节点中的值聚合到驱动器程序中。比如可以用于统计节点中出现错误的行数。

- 全局的，只增不减，记录全局集群的唯一状态
- 在exe中修改它，在driver读取
- executor级别共享的，广播变量是task级别的共享
- 两个application不可以共享累加器，但是同一个app不同的job可以共享


### 广播变量 只读
广播变量可以高效的向所有工作节点发送一个较大的只读值或者表。

spark会自动将闭包中的变量都复制并发送到工作节点上，这样会低效。其实多个工作节点可以共用同一个表变量。

## Checkpoint
checkpoint的本质是将系统运行期键的内存数据结构和状态持久化到硬盘上。需要时可以读取这些数据，构建出来运行时的状态。

checkpointRDD是特殊的RDD，用来从存储体系中恢复检查点的数据。

核心的方法包括：
- writePartitionToCheckpointFile：将RDD分区的数据写入文件。
- writePartitionerToCheckpointDir: 将分区计算器的数据写入文件。
- ReadCheckpointFile: 读文件，返回一个一个迭代器

并且重写了父RDD的方法以实现getPartitions，getPreferredLocations, compute方法用于从checkpoint file中得到数据。

checkpoints前最好对RDD cache下。cache 机制是每计算出一个要 cache 的 partition 就直接将其 cache 到内存了。但 checkpoint 没有使用这种第一次计算得到就存储的方法，而是等到 job 结束后另外启动专门的 job 去完成 checkpoint 。也就是说需要 checkpoint 的 RDD 会被计算两次。因此，在使用 rdd.checkpoint() 的时候，建议加上 rdd.cache()，这样第二次运行的 job 就不用再去计算该 rdd 了，直接读取 cache 写磁盘。

### checkpoint和persist的区别
`persist`虽然可以将 RDD 的 partition 持久化到磁盘，但该 partition 由 blockManager 管理。一旦 driver program 执行结束，也就是 executor 所在进程 CoarseGrainedExecutorBackend stop，blockManager 也会 stop，被 cache 到磁盘上的 RDD 也会被清空（整个 blockManager 使用的 local 文件夹被删除）。

而 checkpoint 将 RDD 持久化到 HDFS 或本地文件夹，如果不被手动 remove 掉，是一直存在的，也就是说可以被下一个 driver program 使用，而 cached RDD 不能被其他 dirver program 使用。


# Shuffle
某种具有共同特征的数据汇聚到一个计算节点上进行计算。一定会发生数据落盘。

Spark中，Shuffle是指将数据重新分区（partition）和重新组合的过程。比如遇到repartition，join，各种ByKey（reduceByKey）。

Shuffle操作涉及以下几个主要步骤：

shuffle写：在Shuffle阶段，Spark将根据键（Key）对数据进行重新分区，将具有相同键的数据发送到同一分区。这个过程涉及数据的传输和网络通信，因为具有相同键的数据可能来自于不同的分区。

shuffle读：在Reduce阶段，Spark将在每个分区上对相同键的数据进行聚合、排序或其他操作，以生成最终结果。
## shuffle write
最开始1.1之前使用的是Hash Based Shuffle，这方法会导致大量的碎片文件，对于系统的IO压力很大。因此在1.2以后改成了Sort Based Shuffle。

关于hash的参考这里[HashShuffleManager](#hashshufflemanager)

关于sort参考这里[sortShuffleManager](#sortshufflemanager)。

>BypassMergeSortShuffleWriter

这个方法不在map段持久化之间进行排序，聚合。

简单来说，`BypassMergeSortShuffleWriter`和Hash Shuffle中的HashShuffleWriter实现基本一致， 唯一的区别在于，map端的多个输出文件会被汇总为一个文件。 所有分区的数据会合并为同一个文件，会生成一个索引文件，是为了索引到每个分区的起始地址，可以随机 access 某个partition的所有数据。


`BypassMergeSortShuffleWriter`原理：给每个分区分配一个临时文件，对每个 record的key 使用分区器（模式是hash，如果用户自定义就使用自定义的分区器）找到对应分区的输出文件句柄，直接写入文件，没有在内存中使用 buffer。 最后copyStream方法把所有的临时分区文件拷贝到最终的输出文件中，并且记录每个分区的文件起始写入位置，把这些位置数据写入索引文件中。

>SortShuffleWriter

可以在map段聚合也不可以不聚合。最后也会有一个分区数据文件和一个分区索引文件

对于`SortShuffleWriter`，具体的处理步骤就是

- 使用 `PartitionedAppendOnlyMap`(对AppendOnlyMap的继承拓展) 或者 `PartitionedPairBuffer` 在内存中进行排序，  排序的 K 是（partitionId， hash（key）） 这样一个元组。
- 如果超过内存 limit， 我 spill 到一个文件中，这个文件中元素也是有序的，首先是按照 partitionId的排序，如果 partitionId 相同， 再根据 hash（key）进行比较排序
- 如果需要输出全局有序的文件的时候，就需要对之前所有的输出文件 和 当前内存中的数据结构中的数据进行  merge sort， 进行全局排序

`SortShuffleWriter` 中使用 `ExternalSorter` 来对内存中的数据进行排序，ExternalSorter内部维护了两个集合`PartitionedAppendOnlyMap`、`PartitionedPairBuffer`， 两者都是使用了 hash table 数据结构。 如果需要进行 aggregation， 就使用 PartitionedAppendOnlyMap（支持 lookup 某个Key，如果之前存储过相同key的K-V 元素，就需要进行 aggregation，然后再存入aggregation后的 K-V）， 否则使用 PartitionedPairBuffer（只进行添K-V 元素）。

纯内存：`AppendOnlyMap`，只是有KV，没有partition信息。


>UnsafeShuffleWriter

底层使用`ShuffleExternalSorter`作为外部排序，因此不能聚合(没有map结构进行存储，无法下map端aggregate。更深层次的原因是不会进行反序列化)。使用了Tungsten内存（即可能是JVM内存，也可能是操作系统的内存）作为缓存，提高了写入性能。
### ExternalSorter
外部排序器可以对map任务的输出在map或者reduce侧进行排序。并将结果写入磁盘

内存＋磁盘：`ExternalAppendOnlyMap` 结构， 这个就是内存中装不下所有元素，就涉及到外部排序。大概的思路就是进行多个堆的有序合并，实现aggregate。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7922c18ce134b4d98ac3dcbc54e01ba.png)


> shuffleExternalSorter
专门对shuffle数据进行排序的外部排序器，将map任务的输出存储到Tungsten；超出limit时溢写到磁盘。与ExternalSorter不同，shuffleExternalSorter没有实现数据持久化，是用调用者UnsafeShuffleWriter实现的。

## shuffle read

shuffle read，通常就是一个stage刚开始时要做的事情。stage的每一个task就需要将上一个stage的计算结果中的所有相同key，从各个节点上 通过网络 拉取到自己所在的节点上，然后进行key的聚合或连接等操作。

由于shuffle write的过程中，task给Reduce端的stage的每个task都创建了一个磁盘文件，因此shuffle read的过程中，每个task只要从上游stage的所有task所在节点上，拉取属于自己的那一个磁盘文件即可。

shuffle read的拉取过程是一边拉取一边进行聚合的。每个shuffle read task都会有一个自己的buffer缓冲，每次都只能拉取与buffer缓冲相同大小的数据，然后通过内存中的一个Map进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到buffer缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。

Shuffle Read 操作发生在 ShuffledRDD#compute 方法中，意味着 Shuffle Read可以发生 ShuffleMapTask 和 ResultTask 两种任务中

> 工作流程

- 首先，shuffle类型的RDD的compute会通过ShuffleManager获取BlockStoreShuffleReader

- 通过BlockStoreShuffleReader中的read方法进行数据的读取，
  
  - 构建了一个ShuffleBlockFetcherIterator
  - 通过mapOutputTracker组件获取每个分区对应的数据block的物理位置
  一个reduce端分区的数据一般会依赖于所有的map端输出的分区数据，所以数据一般会在多个executor(注意是executor节点，通过BlockManagerId唯一标识，一个物理节点可能会运行多个executor节点)节点上，而且每个executor节点也可能会有多个block，在shuffle写过程的分析中我们也提到，每个map最后时输出一个数据文件和索引文件，也就是一个block，但是因为一个节点

- 这个方法通过ShuffleBlockFetcherIterator对象封装了远程拉取数据的复杂逻辑，并且最终将拉取到的数据封装成流的迭代器的形式。

- 对所有的block的流进行层层装饰，包括反序列化，任务度量值（读入数据条数）统计。
- 对数据进行聚合
- 对聚合后的数据进行排序

分为本地读取和远端读取：本地读取比较简单，主要就是通过本节点的`BlockManager`来获取块数据，并通过索引文件获取数据指定分区的数据。

远端读取时通过网络读取其他executor上的数据，因为网络读取，所以每次最多到5个节点上读取并且请求大小不超过48M的5分之一。每次都构建一个fetchRequest，去请求拉取数据。

## shuffle优化

节点本地性（Node Locality）：Spark尽可能地在同一节点上进行Shuffle操作，以减少数据在网络中的传输。这可以通过合理的分区策略和任务调度来实现。

压缩（Compression）：Spark支持在Shuffle过程中对数据进行压缩，减少网络传输的数据量，从而提高性能和效率。

聚合（Aggregation）：当可能时，Spark会在Map阶段对具有相同键的数据进行本地聚合，以减少Shuffle阶段的数据量。

Sort-based Shuffle：Spark默认使用Sort-based Shuffle算法来进行数据的重新分区和排序。这种算法在性能和内存使用之间进行平衡，能够有效地处理大规模数据。

## partitioner
Partitioner只对键值对类型的RDD有效，即PairRDD（例如(key, value)形式的RDD）

Partitioner是用于对RDD进行分区的对象，它定义了数据在分布式环境中如何划分和分配到不同的计算节点上。Spark提供了几种内置的Partitioner，包括：

- HashPartitioner（哈希分区器）：根据键的哈希值进行分区。默认情况下，Spark的groupByKey和reduceByKey等操作会使用HashPartitioner。弊端是数据不均匀，容易导致数据倾斜

- RangePartitioner（范围分区器）：根据键的排序范围进行分区。适用于已经排序的数据集，例如sortByKey操作。

- Custom Partitioner（自定义分区器）：可以根据特定需求自定义分区逻辑的分区器。通过继承org.apache.spark.Partitioner类并实现必要的方法来创建自定义分区器。

## 聚合类算子

尽量避免使用shuffle类算子。也就是将分布在集群中多个节点上的同一个key，拉取到同一个节点上，进行聚合或join等操作。比如reduceByKey、join、distinct、repartition等算子。尽量使用map类的非shuffle算子。
# Spark的Join实现
Spark提供了五种执行Join操作的机制，分别是：


- Broadcast Hash Join: 适用于（等值）小表Join。将小表先发送给 driver，然后由 driver 统一广播给大表分区所在的所有 executor 节点。每个excecutor进行hash join。由小表对应的 buildIter 构建 hash table，大表对应的 streamIter 来探测。
  - 条件是小表必须小于参数 spark.sql.autoBroadcastJoinThreshold，该值默认为 10M，或者指定了 broadcast hint。

- Shuffle Hash Join：大表join稍微小的表无排序。将两张表按 join key 重新分区shuffle（两表相同的 key 的数据会落在同一节点上），数据会重新分布到集群中的所有节点上。每个executor执行hash join，再将各个分区的数据进行合并。
  - 条件：`spark.sql.join.prefersortmergeJoin`设置为false。左表未排序。
- Sort Merge Join：两张大表进行JOIN时，使用该方式。
  - 原理：1. shuffle：将两张大表根据 join key 重新分区，这样两张表的数据将会分布到整个集群，以便更大程度的并行进行下一步操作。2. sort：对每个分区节点内的两表数据，分别排序。3. merge：对两表进行 join 操作。各分区各自遍历自己的 streamIter，对于每条记录，使用顺序查找的方式从 buildIter 查找对应的记录，由于两个表都是排序的，每次处理完 streamIter 的一条记录后，对于 streamIter 的下一条记录，从 buildIterm 上一次查找结束的位置开始查找即可，而不需要重新再开始查找。
- Cartesian Join： cross join
- Broadcast Nested Join：默认兜底方法

当都可以时：
Broadcast Hash Join > Sort Merge Join > Shuffle Hash Join > Cartesian Join。

简单来说就是：
- 大表和大表：shuffle join。每个节点都要和其他节点通讯，并根据哪个节点有你当前用于join的key来传输数据。因此网络负担比较重。
- 大表与小表：broadcast join。可以把数据量小的df复制到集群的所有工作节点。然后每个工作节点单独执行，不需要额外的通信。


Spark支持三种Join实现方式：shuffle hash join、broad hash join、sort merge join。

## 如何选择join机制
参数配置，hint参数(手动指定方法),输入数据集大小,Join类型,Join条件
`df1.hint("broadcast").join(df2, ...)`

>原理
通过 join key 和逻辑计划的大小来选择合适的 join 方法。 
- 等值join：

  - Broadcast：如果某一边表小于 spark.sql.autoBroadcastJoinThreshold 或显示指定 broadcast hint（如用户使用 DataFrame 的 org.apache.spark.sql.functions.broadcast() 方法）。
  - 否则使用Shuffle hash join：如果表的每个分区能构建为 hash table。
  - 否则使用Sort merge join：如果匹配的 join key 是能排序的

- 非等值
  - BroadcastNestedLoopJoin：如果某一边的表能被 broadcast
  - CartesianProduct：用于 Inner Join
  - BroadcastNestedLoopJoin：可能会 OOM，但没其他选择了。

## shuffle hash join/ sort merge Join
>Shuffle Hash Join的基本步骤主要有以下两点：

首先，对于两张参与JOIN的表，分别按照join key进行重分区，该过程会涉及Shuffle，其目的是将相同join key的数据发送到同一个分区，方便分区内进行join。

其次，对于每个Shuffle之后的分区，会将小表的分区数据构建成一个Hash table，然后根据join key与大表的分区数据记录进行匹配。

>Sort Merge Join主要包括三个阶段：

- Shuffle Phase : 两张大表根据Join key进行Shuffle重分区
- Sort Phase: 每个分区内的数据进行排序
- Merge Phase: 对来自不同表的排序好的分区数据进行JOIN，通过遍历元素，连接具有相同Join key值的行来合并数据集

可以看出，无论分区有多大，Sort Merge Join都不用把某一侧的数据全部加载到内存中，而是即用即取即丢，从而大大提升了大数据量下sql join的稳定性。


https://qileq.com/tech/spark/sql/join/
# Spark算子类型
从大方向来说，Spark 算子大致可以分为以下两类:transform 算子，action算子。

从小方向来说，Spark 算子大致可以分为以下三类:
- Value数据类型的Transformation算，针对处理的数据项是Value型的数据。
- Key-Value数据类型的Transfromation算子，针对处理的数据项是Key-Value型的数据对。
- Action算子，这类算子会触发SparkContext提交Job作业。
  
## transform 算子
这种变换并不触发提交作业，完成作业中间过程处理。Transformation 操作是延迟计算的，也就是说从一个RDD 转换生成另一个 RDD 的转换操作不是马上执行，需要等到有 Action 操作的时候才会真正触发运算
## action算子
这类算子会触发 SparkContext 提交 Job 作业

## 算子例子
- Value数据类型的Transformation算子
  - 输入分区与输出分区一对一型：**map算子，flatMap算子，mapPartitions算子**，glom算子
  - 输入分区与输出分区多对一型：union算子，cartesian算子
  - 输入分区与输出分区多对多型：**groupBy算子**
  - 输出分区为输入分区子集型：filter算子，**distinct算子**，subtract算子，sample算子，takeSample算子
  - Cache型：**cache算子，persist算子**

- Key-Value数据类型的Transfromation算子
  - 输入分区与输出分区一对一：mapValues算子
  - 对单个RDD聚集：combineByKey算子，reduceByKey算子，partitionBy算子
  - 两个RDD聚集：Cogroup算子
- 连接：join算子，leftOutJoin和 rightOutJoin算子

- Action算子
  - 无输出：foreach算子
  - HDFS：saveAsTextFile算子，saveAsObjectFile算子
  - Scala集合和数据类型：collect算子，collectAsMap算子，reduceByKeyLocally算子，lookup算子，count算子,top算子，reduce算子，fold算子，aggregate算子
# 与MR对比
## 优点
Spark速度更快，因为可以使用多线程并且可以使用内存加速计算。

1. 减少磁盘I/O: MR中他，map端将中间输出和结果存储在磁盘中，reduce端又需要从磁盘读写中间结果，容易导致IO瓶颈。 Spark使用了内存存储中间结果。
2. 增加并行度：中间结果写入磁盘和从磁盘读取中间结果是不同环节。hadoop只能串行，spark可以串行也可以并行。
3. 避免重新计算：stage中某个分区的task失败以后，会重新对此stage调度，但是在重新调度的时候会过滤掉已经成功执行的分区任务，避免重复计算。
4. 可选的shuffle排序：MapReduce在shuffle前有固定的排序操作，但是spark可以根据不同的场景选在map端还是在reduce端。
# Spark调优
总体的思路包括，开发调优、资源调优、数据倾斜调优、shuffle调优。
## 开发调优——（适当持久化，shuffle类算子使用优化，broadcast，更好的算子使用）
1. 开发时，对于同一份数据源，只应该创建一个RDD，不能创建多个RDD来代表同一份数据。
2. 对多次使用的RDD进行持久化，将RDD中的数据保存到内存或者磁盘中。保证对一个RDD执行多次算子操作时，这个RDD本身仅仅被计算一次。
  Spark中对于一个RDD执行多次算子的默认原理是这样的：每次对一个RDD执行一个算子时，都会重新从源头计算一遍得到RDD，然后再对这个RDD执行你的算子操作。这种方式的性能是很差的。

`cache()`会尝试把数据全部持久化到内存中，`persist()`可以手动选择持久化方法。

3. 尽量避免使用shuffle类算子。也就是将分布在集群中多个节点上的同一个key，拉取到同一个节点上，进行聚合或join等操作。比如reduceByKey、join、distinct、repartition等算子。尽量使用map类的非shuffle算子。
  
    一个可能的思路，对小表进行broadcast，进而避免shuffle join。

4. 广播外部变量
   遇到需要在算子函数中使用**外部变量**的场景（尤其是大变量，比如100M以上的大集合），那么此时就应该使用Spark的广播（Broadcast）功能来提升性能。

    在算子函数中使用到外部变量时，默认情况下，Spark会将该变量复制多个副本，通过网络传输到**task**中，此时每个task都有一个变量副本。如果变量本身比较大的话（比如100M，甚至1G），那么大量的变量副本在网络中传输的性能开销，以及在各个节点的Executor中占用过多内存导致的频繁GC，都会极大地影响性能。

    广播后的变量，会保证**每个Executor的内存中，只驻留一份变量副本**。在算子函数用广播变量时，首先会判断当前task所在Executor内存中，是否有变量副本。如果有则直接使用；如果没有则从Driver或者其他Executor节点上远程拉取一份放到本地Executor内存中。而Executor中的task执行时共享该Executor中的那份变量副本。这样的话，可以大大减少变量副本的数量，从而**减少网络传输的性能开销，并减少对Executor内存的占用开销，降低GC的频率**。

5. 使用更好的算子：
   1. 使用reduceByKey/aggregateByKey替代groupByKey，因为可以有预聚合。
   2. 使用mapPartitions替代普通map。可以一次性处理整个分区的数据，但是需要注意kennel导致OOM。
   3. 使用foreachPartitions替代foreach。可以批量的处理，可以降低一些开销，比如数据库的连接之类的。
   4. 使用filter之后进行coalesce操作。手动减少RDD的partition数量，将RDD中的数据压缩到更少的partition中去。
  
  ## 资源调优
- **num-executors**：设置Spark作业总共要用多少个Executor进程来执行。Driver在向YARN集群管理器申请资源时，YARN集群管理器会尽可能按照你的设置来在集群的各个工作节点上，启动相应数量的Executor进程。设置的太少，无法充分利用集群资源；设置的太多的话，大部分队列可能无法给予充分的资源。
- `executor-memory`：每个Executor进程的内存。Executor内存的大小，很多时候直接决定了Spark作业的性能，而且跟常见的JVM OOM异常，也有直接的关联。num-executors乘以executor-memory，是不能超过队列的最大内存量的。
- `executor-cores`：每个Executor进程的CPU core数量。这个参数决定了每个Executor进程并行执行task线程的能力。因为每个CPU core同一时间只能执行一个task线程，因此每个Executor进程的CPU core数量越多，越能够快速地执行完分配给自己的所有task线程。可以和物理机器的内存和core比例匹配。
- `driver-memory`：Driver进程的内存。如果需要使用collect算子将RDD的数据全部拉取到Driver上进行处理，那么必须确保Driver的内存足够大，否则会出现OOM内存溢出的问题。
- **spark.default.parallelism**：500～1000 设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的Spark作业性能。如果不去设置这个参数，那么此时就会导致Spark自己根据底层HDFS的block数量来设置task的数量，默认是一个HDFS block对应一个task。Spark官网建议的设置原则是，设置该参数为num-executors * executor-cores的2~3倍较为合适
- `spark.storage.memoryFraction`：设置RDD持久化数据在Executor内存中能占的比例，默认是0.6。如果Spark作业中，有较多的**RDD持久化操作**，该参数的值可以适当提高一些，保证持久化的数据能够容纳在内存中。避免内存不够缓存所有的数据，导致数据只能写入磁盘中，降低了性能。但是如果Spark作业中的**shuffle类操作比较多**，而持久化操作比较少，那么这个参数的值适当降低一些比较合适。此外，如果发现作业由于**频繁的gc**导致运行缓慢（通过spark web ui可以观察到作业的gc耗时），意味着**task执行用户代码的内存**不够用，那么同样建议调低这个参数的值。
- `spark.shuffle.memoryFraction`：shuffle过程中一个task拉取到上个stage的task的输出后，进行聚合操作时能够使用的Executor内存的比例，默认是0.2。shuffle操作在进行聚合时，如果发现使用的内存超出了这个20%的限制，那么多余的数据就会溢写到磁盘文件中去，此时就会极大地降低性能。此外，如果发现作业由于频繁的gc导致运行缓慢，意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。



## 数据倾斜调优
### 现象和原理
Spark作业看起来会运行得非常缓慢，甚至可能因为某个task处理的数据量过大导致内存溢出。
>数据倾斜出现的场景：

- 绝大多数task执行得都非常快，但个别task执行极慢。比如，总共有1000个task，997个task都在1分钟之内执行完了，但是剩余两三个task却要一两个小时。这种情况很常见。
- 原本能够正常执行的Spark作业，某天突然报出OOM（内存溢出）异常，观察异常栈，是我们写的业务代码造成的。这种情况比较少见。

>数据倾斜发生的原理:

在进行shuffle的时候，必须将各个节点上相同的key拉取到某个节点上的一个task来进行处理。比如按照key进行聚合或join等操作。此时如果某个key对应的数据量特别大的话，就会发生数据倾斜。比如大部分key对应10条数据，但是个别key却对应了100万条数据，那么大部分task可能就只会分配到10条数据，然后1秒钟就运行完了；但是个别task可能分配到了100万数据，要运行一两个小时。因此，整个Spark作业的运行进度是由运行时间最长的那个task决定的。


>定位问题的方法

通过4040端口的webUI去查看哪个阶段卡了，是哪一哥算子发生了问题。
### 优化方案——数据倾斜 / join调优
1. 预处理数据：在spark之前处理数据。保证spark运行效率。
2. 过滤少数倾斜Key：某些key异常的数据较多。可以动态判定哪些key的数据量最多然后再进行过滤，那么可以使用sample算子对RDD进行采样，然后计算出每个key的数量，取数据量最多的key过滤掉即可。
3. 提高shuffle并行度：`spark.sql.shuffle.partitions`代表了shuffle read task的并行度，默认是200比较小。增加shuffle read task的数量，可以让原本分配给一个task的多个key分配给多个task，从而让每个task处理比原来更少的数据。（方案对于某个key倾斜不适用，因为这个key还是被打到了某一个task）
4. Join使用broadcast/reduce类使用随机前缀打散key：对于join改成broadcast实际上是避免了shuffle。reduce方法实际上是降低了某一个key的数据量。用map方法替代了reduce方法。
5. 两个表都很大的join，可以打散+聚合。 


核心思想就是把key分配到不同分区执行。


## shuffle调优
大多数Spark作业的性能主要就是消耗在了shuffle环节，因为该环节包含了大量的磁盘IO、序列化、网络数据传输等操作。

有两种shuffle，1.2以前默认的是HashShuffleManager，HashShuffleManager有着一个非常严重的弊端，就是会产生大量的中间磁盘文件，进而由大量的磁盘IO操作影响了性能。

后面变成了SortShuffleManager，这个对于shuffle有所改进，对于临时文件会最后合并为一个文件，因此每个task就只会有一个磁盘文件。

### HashShuffleManager
#### shuffleWrite/shuffleRead
**shuffle write**阶段，主要就是在一个stage结束计算之后，为了下一个stage可以执行shuffle类的算子（比如reduceByKey），而将每个task处理的数据按key进行“分类”。所谓“分类”，就是对相同的key执行hash算法，从而将相同key都写入同一个磁盘文件中，而每一个磁盘文件都只属于下游stage的一个task。在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去。

>那么每个执行shuffle write的task，要为下一个stage创建多少个磁盘文件呢？很简单，**下一个stage的task有多少个，当前stage的每个task就要创建多少份磁盘文件**。比如下一个stage总共有100个task，那么当前stage的每个task都要创建100份磁盘文件。如果当前stage有50个task，总共有10个Executor，每个Executor执行5个Task，那么每个Executor上总共就要创建500个磁盘文件，所有Executor上会创建5000个磁盘文件。由此可见，未经优化的shuffle write操作所产生的磁盘文件的数量是极其惊人的。

shuffle read，通常就是一个stage刚开始时要做的事情。此时该stage的每一个task就需要将上一个stage的计算结果中的所有相同key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行key的聚合或连接等操作。由于shuffle write的过程中，task给下游stage的每个task都创建了一个磁盘文件，因此shuffle read的过程中，**每个task只要从上游stage的所有task所在节点上，拉取属于自己的那一个磁盘文件即可**。

shuffle read的拉取过程是一边拉取一边进行聚合的。每个shuffle read task都会有一个自己的buffer缓冲，**每次都只能拉取与buffer缓冲相同大小的数据**，然后通过内存中的一个Map进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到buffer缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。

**HashShuffleManager可以进行优化，实现复用磁盘文件**。假设第一个stage有50个task，第二个stage有100个task，总共还是有10个Executor，每个Executor执行5个task。那么原本使用未经优化的HashShuffleManager时，每个Executor会产生500个磁盘文件，所有Executor会产生5000个磁盘文件的。但是此时经过优化之后，每个Executor创建的磁盘文件的数量的计算公式为：CPU core的数量（1） * 下一个stage的task数量。也就是说，每个Executor此时只会创建100个磁盘文件（下一个stage的task数量），所有Executor只会创建1000个磁盘文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/99624383f2f748d49a404f1133e5ce57.png)

## sortShuffleManager
SortShuffleManager的运行机制主要分成两种，一种是普通运行机制，另一种是bypass运行机制。当shuffle read task的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值时（默认为200），就会启用bypass机制。

>普通运行机制

在该模式下，数据会先写入一个内存数据结构中，此时根据不同的shuffle算子，可能选用不同的数据结构。如果是reduceByKey这种聚合类的shuffle算子，那么会选用Map数据结构，一边通过Map进行聚合，一边写入内存；如果是join这种普通的shuffle算子，那么会选用Array数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。

在溢写到磁盘文件之前，**会先根据key对内存数据结构中已有的数据进行排序**。排序过后，会分批将数据写入磁盘文件。默认的batch数量是10000条，也就是说，排序好的数据，会以每批1万条数据的形式分批写入磁盘文件。

一个task将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是merge过程，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中。此外，由于一个task就只对应一个磁盘文件，也就意味着该task为下游stage的task准备的数据都在这一个文件中，因此还会单独写一份索引文件，其中标识了下游各个task的数据在文件中的start offset与end offset。

SortShuffleManager由于有一个磁盘文件merge的过程，因此大大减少了文件数量。比如第一个stage有50个task，总共有10个Executor，每个Executor执行5个task，而第二个stage有100个task。由于每个task最终只有一个磁盘文件，因此此时每个Executor上只有5个磁盘文件(因为当前只有5个task执行)，所有Executor只有50个磁盘文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/573e37543a0b4a989828fbf0da45ff85.png)

> bypass：不会对数据真的排序 

bypass运行机制的触发条件如下： 
* shuffle map task数量小于spark.shuffle.sort.bypassMergeThreshold参数的值。 
* 不是聚合类的shuffle算子（比如reduceByKey就是聚合类的算子）。

此时task会为每个下游task都创建一个临时磁盘文件，并将数据按**将key写入对应的磁盘文件之中**。当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

该过程的磁盘写机制其实跟未经优化的HashShuffleManager是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。因此少量的最终磁盘文件，也让该机制相对未经优化的HashShuffleManager来说，shuffle read的性能会更好。

而该机制与普通SortShuffleManager运行机制的不同在于：第一，磁盘写机制不同；第二，不会进行排序。也就是说，启用该机制的最大好处在于，shuffle write过程中，不需要进行数据的排序操作，也就节省掉了这部分的性能开销。

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c2eb2569bbe4a778deea94f63c65dbb.png)

相关参数：https://tech.meituan.com/2016/05/12/spark-tuning-pro.html

# spark 3.0
从 Spark 3.0 官方的Release Notes可以看到，这次大版本的升级主要是集中在性能优化和文档丰富上，其中 46%的优化都集中在 Spark SQL 上。


- 自适应查询执行Adaptive Query Execution (AQE)
AQE 对于整体的 Spark SQL 的执行过程做了相应的调整和优化(如下图)，它最大的亮点是可以根据已经完成的计划结点真实且精确的执行统计结果来不停的反馈并重新优化剩下的执行计划。

  - 自动调整 reducer 的数量，减小 partition 数量：AQE 能够很好的解决这个问题，在 reducer 去读取数据时，会根据用户设定的分区数据的大小(spark.sql.adaptive.advisoryPartitionSizeInBytes)来自动调整和合并(Coalesce)小的 partition，自适应地减小 partition 的数量，以减少资源浪费和 overhead，提升任务的性能。参考示例图中可以看到从最开始的 shuffle 产生 50 个 partitions，最终合并为只有 5 个 partitions。
  - 自动解决 Join 时的数据倾斜问题：AQE 由于可以实时拿到运行时的数据，通过 Skew Shuffle Reader 自动调整不同 key 的数据大小(spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes) 来避免数据倾斜，从而提高性能。参考示例图可以看到 AQE 自动将 A 表里倾斜的 partition 进一步划分为 3 个小的 partitions 跟 B 表对应的 partition 进行 join，消除短板倾斜任务
  - 优化 Join 策略：AQE 可以在 Join 的初始阶段获悉数据的输入特性，并基于此选择适合的 Join 算法从而最大化地优化性能。比如从 Cost 比较高的 SortMerge 在不超过阈值的情况下调整为代价较小的 Broadcast Join。
- 动态分区修剪Dynamic Partition Pruning(DPP)：DPP 主要解决的是对于星型模型的查询场景中过滤条件无法下推的情况。通过 DPP 可以将小表过滤后的数据作为新的过滤条件下推到另一个大表里，从而可以做到对大表 scan 运行阶段的提前过滤掉不必要 partition 读取。这样也可以避免引入不必要的额外 ETL 过程（例如预先 ETL 生成新的过滤后的大表），在查询的过程中极大的提升查询性能

# Spark sql
Spark SQL是Apache Spark中的一个模块，用于处理结构化数据并提供SQL查询的功能。它提供了一种编程接口，可以使用SQL语句或DataFrame/Dataset API来查询和操作数据。

Spark SQL的主要特性包括：

- SQL查询：Spark SQL允许使用标准的SQL语句来查询数据。它支持多种数据源（如Hive、JSON、Parquet、Avro等），并提供了强大的查询优化和执行引擎，以便高效地执行查询操作。

- DataFrame和Dataset API：Spark SQL引入了DataFrame和Dataset API，这是一种面向对象的API，用于以类型安全的方式操作结构化数据。它提供了丰富的操作和转换方法，用于处理数据、进行聚合、过滤和连接等操作。

- 查询优化和执行引擎：Spark SQL具有查询优化器和执行引擎，它可以对查询进行优化，包括谓词下推、列剪裁、表达式重写等技术，以提高查询性能和效率。
- 集成Hive和现有系统：如果已经使用Hive作为数据仓库或有现有的Hive表结构，那么使用Spark SQL能够无缝集成Hive，直接读取和写入Hive表，并共享Hive的元数据、UDFs等。

对于结构化数据，并且主要做数据转换、过滤、聚合等操作，那么使用Spark SQL更简洁。Spark SQL提供了DataFrame和Dataset API，使得结构化数据处理更易于编写和维护。

## 执行流程
spark sql的架构如图。SQL语句，经过一个优化器（Catalyst），转化为RDD，交给集群执行
![在这里插入图片描述](https://img-blog.csdnimg.cn/96841cb727384e6b9e3bf7132aa02d7b.png)

其中核心是 Catalyst优化器。sql语句的本质就是，解析（Parser）、优化（Optimizer）、执行（Execution）。

1. Parser阶段：将SQL语句解析成一颗语法树。使用第三方类库ANTLR进行实现。的（包括我们熟悉的Hive、Presto、SparkSQL等都是由ANTLR实现的）。在这个过程中，会判断SQL语句是否符合规范。不会对表名，表字段进行检查。
2. Analyzer阶段：生成逻辑计划。此时元数据信息包括两部分：表的Scheme信息，如列名、数据类型、表的物理位置等，和基本函数信息。此过程就会判断SQL语句的表名，字段名是否真的在元数据库里存在。
3. Optimizer模块：优化逻辑计划。Optimizer优化模块是整个Catalyst的核心，上面提到优化器分为基于规则的优化（RBO）和基于代价优化（CBO）两种。
   1. 基于规则的优化策略实际上就是对语法树进行一次遍历，在进行相应的等价转换。
      - **谓词下推**(先filter再join) 、
      - **常量累加**(x+(100+80)->x+180) 、
      - **列值裁剪**(当用到一个表时，不需要扫描它的所有列值，而是扫描只需要的id，不需要的裁剪掉。这一优化一方面大幅度减少了网络、内存数据量消耗，另一方面对于列式存储数据库来说大大提高了扫描效率。) 
   
   2. 生成多个可以执行的物理计划Physical Plan；接着CBO（基于代价优化）优化策略会根据Cost Model算出每个Physical Plan的代价，并选取代价最小的 Physical Plan作为最终的Physical Plan。
   
https://cloud.tencent.com/developer/article/2008340
# 其他问题
## spark的小文件问题
spark的小文件问题会给分别给读写两端带来问题。对于读来说，处理一大堆小文件的读入会导致每个rdd的partition数量太多。由于每个rdd的partition对应一个task，因此会产生太多的task，从而增加了网络通信和任务调度的开销。一般在读入数据以后，我们可以repartition为min(task_num, 200).

对于写来说，推荐是每个文件的大小为128MB左右。因此我们在最终result 写之前可以使用repartition或coalesce方法将数据重新分区
### coalease和repartition的区别
coalesce适用于减小分区数量，这个方法不会引发shuffle，利用已有的partition去尽量减少分区数。repartition创建新的partition并且使用 full shuffle。 

repartition只是coalesce接口中shuffle为true的实现


    T表有10G数据  有100个partition 资源也为--executor-memory 2g --executor-cores 2 --num-executors 5。我们想要结果文件只有一个
    1. 如果用coalesce：sql(select * from T).coalesce(1)
        5个executor 有4个在空跑，只有1个在真正读取数据执行，这时候效率是极低的。所以coalesce要慎用，而且它还用产出oom问题，这个我们以后再说。
    2. 如果用repartition：sql(select * from T).repartition(1)
        这样效率就会高很多，并行5个executor在跑（10个task）,然后shuffle到同一节点，最后写到一个文件中

## Spark程序执行，有时候默认为什么会产生很多task，怎么修改默认task执行个数？

- 有很多小文件的时候，有多少个输入block就会有多少个task启动
- spark中有partition的概念，每个partition都会对应一个task，task越多，在处理大规模数据的时候，就会越有效率
## 为什么shuffle性能差/shuffle发生了什么
1. 各个节点上的相同key都会先写入本地磁盘文件中，然后
2. 其他节点需要通过网络传输拉取各个节点上的磁盘文件中的相同key。
3. 相同key都拉取到同一个节点进行聚合操作时，还有可能会因为一个节点上处理的key过多，导致内存不够存放，进而溢写到磁盘文件中。
 
因此在shuffle过程中，可能会发生大量的**磁盘文件读写的IO操作**，以及**数据的网络传输操作**。磁盘IO和网络数据传输也是shuffle性能较差的主要原因。
## reduceByKey和groupByKey的区别
reduceByKey是按照key在shuffle之前有预聚合，返回的是RDD[K,V]，groupByKey是按照key进行分组直接shuffle。两者相比reduceByKey性能更好。
## spark的序列化问题
在Spark中，主要有三个地方涉及到了序列化： 
* 在算子函数中使用到外部变量时，该变量会被序列化后进行网络传输
*  将自定义的类型作为RDD的泛型类型时（比如JavaRDD，Student是自定义类型），所有自定义类型对象，都会进行序列化。因此这种情况下，也要求自定义的类必须实现Serializable接口。 
* 使用可序列化的持久化策略时（比如MEMORY_ONLY_SER），Spark会将RDD中的每个partition都序列化成一个大的字节数组。

对于这三种出现序列化的地方，我们都可以通过使用Kryo序列化类库，来优化序列化和反序列化的性能。Spark默认使用的是Java的序列化机制，也就是ObjectOutputStream/ObjectInputStream API来进行序列化和反序列化。但是Spark同时支持使用Kryo序列化库，Kryo序列化类库的性能比Java序列化类库的性能要高很多。

## 描述一下spark的运行过程和原理
我们使用spark-submit提交一个Spark作业之后，这个作业就会启动一个对应的Driver进程。根据你使用的部署模式（deploy-mode）不同，Driver进程可能在本地启动，也可能在集群中某个工作节点上启动。

而Driver进程要做的第一件事情，就是向集群管理器（可以是Spark Standalone集群，也可以是其他的资源管理集群）申请运行Spark作业需要使用的资源，这里的资源指的就是Executor进程。YARN集群管理器会根据我们为Spark作业设置的资源参数，在各个工作节点上，启动一定数量的Executor进程，每个Executor进程都占有一定数量的内存和CPU core。

在申请到了作业执行所需的资源之后，Driver进程就会开始调度和执行我们编写的作业代码了。Driver进程会将我们编写的Spark作业代码分拆为多个stage，每个stage执行一部分代码片段，并为每个stage创建一批task，然后将这些task分配到各个Executor进程中执行。task是最小的计算单元，负责执行一模一样的计算逻辑（也就是我们自己编写的某个代码片段），**只是每个task处理的数据不同而已**。一个stage的所有task都执行完毕之后，会在各个节点**本地的磁盘文件**中写入计算中间结果，然后Driver就会调度运行下一个stage。下一个stage的task的输入数据就是上一个stage输出的中间结果。如此循环往复，直到将我们自己编写的代码逻辑全部执行完，并且计算完所有的数据，得到我们想要的结果为止。

Spark是根据shuffle类算子来进行stage的划分。如果我们的代码中执行了某个shuffle类算子（比如reduceByKey、join等），那么就会在该算子处，划分出一个stage界限来。可以大致理解为，shuffle算子执行之前的代码会被划分为一个stage，shuffle算子执行以及之后的代码会被划分为下一个stage。因此一个stage刚开始执行的时候，它的每个task可能都会从上一个stage的task所在的节点，去通过网络传输拉取需要自己处理的所有key，然后对拉取到的所有相同的key使用我们自己编写的算子函数执行聚合操作（比如reduceByKey()算子接收的函数）。这个过程就是shuffle。

当我们在代码中执行了cache/persist等持久化操作时，根据我们选择的持久化级别的不同，每个task计算出来的数据也会保存到Executor进程的内存或者所在节点的磁盘文件中。

## Executor的内存/core使用是什么样的
因此Executor的内存主要分为三块：
1. 第一块是让task执行我们自己编写的代码时使用，默认是占Executor总内存的20%；
2. 第二块是让task通过shuffle过程拉取了上一个stage的task的输出后，进行聚合等操作时使用，默认也是占Executor总内存的20%；
3. 第三块是让RDD持久化时使用，默认占Executor总内存的60%。

**task的执行速度是跟每个Executor进程的CPU core数量有直接关系的**。一个CPU core同一时间只能执行一个线程。而每个Executor进程上分配到的多个task，都是以**每个task一条线程**的方式，多线程并发运行的。

## 使用spark实现topN
https://juejin.cn/post/7120786529286357000


基础方法：采用groupByKey。按照key对数据进行聚合（groupByKey）。对同组的key的所有value先转换为List，然后进行排序，最后取TopN

优化就是对于每个partition取topK然后再计算。先获取每个分区的TopN，后获取全局TopN