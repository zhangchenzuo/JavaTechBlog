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
- [spark的内存模型](#spark的内存模型)
- [Job，Stage，Task](#jobstagetask)
  - [DAG](#dag)
  - [partition](#partition)
  - [Job](#job)
  - [stage](#stage)
    - [依赖划分原则](#依赖划分原则)
    - [宽窄依赖](#宽窄依赖)
  - [Task](#task)
- [spark的数据结构](#spark的数据结构)
  - [RDD](#rdd)
    - [容错机制](#容错机制)
    - [RDD持久化](#rdd持久化)
      - [persist/cache](#persistcache)
      - [如何选择一种最合适的持久化策略](#如何选择一种最合适的持久化策略)
      - [两种操作](#两种操作)
  - [DataFrame](#dataframe)
  - [DataSet](#dataset)
  - [三种数据结构的对比](#三种数据结构的对比)
  - [分布共享变量](#分布共享变量)
    - [累加器 只写](#累加器-只写)
    - [广播变量 只读](#广播变量-只读)
- [Shuffle](#shuffle)
  - [shuffle优化](#shuffle优化)
  - [partitioner](#partitioner)
  - [聚合类算子](#聚合类算子)
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
- [其他问题](#其他问题)
  - [Spark程序执行，有时候默认为什么会产生很多task，怎么修改默认task执行个数？](#spark程序执行有时候默认为什么会产生很多task怎么修改默认task执行个数)
  - [为什么shuffle性能差/shuffle发生了什么](#为什么shuffle性能差shuffle发生了什么)
  - [spark的序列化问题](#spark的序列化问题)
  - [描述一下spark的运行过程和原理](#描述一下spark的运行过程和原理)
  - [Executor的内存/core使用是什么样的](#executor的内存core使用是什么样的)

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
本地模式

    Spark不一定非要跑在hadoop集群，可以在本地，起多个线程的方式来指定。方便调试，本地模式分三类
        local：只启动一个executor
        local[k]: 启动k个executor
        local：启动跟cpu数目相同的 executor

standalone模式

    分布式部署集群，自带完整的服务，资源管理和任务监控是Spark自己监控，这个模式也是其他模式的基础

Spark on yarn模式

    分布式部署集群，资源和任务监控交给yarn管理
    粗粒度资源分配方式，包含cluster和client运行模式
        cluster 适合生产，driver运行在集群子节点，具有容错功能
        client 适合调试，dirver运行在客户端

Spark On Mesos模式

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

# spark的内存模型


spark除了把内存作为计算资源以外，还作为了存储资源。MemoryManager负责管理。spark把内存划分为了堆内存和堆外内存。堆内存时JVM堆内存的一部分，堆外内存时工作节点中系统内存的一部分空间。内存可以被分为StorageMemoryPool和ExecutionMemoryPool。


![在这里插入图片描述](https://img-blog.csdnimg.cn/c98eb9ce8394456c80b13607d651a229.png)

# Job，Stage，Task
一个Application由一个Driver和若干个Job构成，一个Job由多个Stage构成，一个Stage由多个没有Shuffle关系的Task组成。
## DAG
用来反映RDD之间的依赖关系。
![在这里插入图片描述](https://img-blog.csdnimg.cn/12d9ed994680447eb0d020aaac7e4a76.png)
## partition

数据分区，一个RDD的数据可以被划分为多少个分区。Spark根据partition的数量来确定Task的数量。

分区太少，会导致较少的并发、数据倾斜、或不正确的资源利用。分区太多，导致任务调度花费比实际执行时间更多的时间。若没有配置分区数，默认的分区数是：所有执行程序节点上的内核总数。

一般一个core上由2-4个分区。
## Job
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

包括的方法：
- partitions: Option[Partitioner]
- getDependencies()
- getPartitions():Array[Partition]
- getPerferrededLocations(split: Partition): Seq[String]
- Map(): MapPartitionsRDD
- count(): Long
- compute(split: Partition, context: TaskContext): Iterator[T]

>弹性

1. 内存的弹性：内存与磁盘的自动切换
2. 容错的弹性：数据丢失可以自动恢复
3. 计算的弹性：计算出错重试机制
4. 分片的弹性：根据需要重新分片 
### 容错机制
RDD的容错，主要是从保存了依赖关系上体现的。

　Spark框架层面的容错机制，主要分为三大层面（调度层、RDD血统层、Checkpoint层），在这三大层面中包括Spark RDD容错四大核心要点。

1. Stage输出失败，上层调度器DAGScheduler重试。
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


#### 两种操作
- action：对RDD真正执行计算。collect、reduce、foreach、count
- Transformation：对RDD进行转换
- Controller：对性能效率和容错方面的支持。persist , cache, checkpoint
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


# Shuffle
某种具有共同特征的数据汇聚到一个计算节点上进行计算

Spark中，Shuffle是指将数据重新分区（partition）和重新组合的过程。

Shuffle操作涉及以下几个主要步骤：

shuffle写：在Shuffle阶段，Spark将根据键（Key）对数据进行重新分区，将具有相同键的数据发送到同一分区。这个过程涉及数据的传输和网络通信，因为具有相同键的数据可能来自于不同的分区。

shuffle读：在Reduce阶段，Spark将在每个分区上对相同键的数据进行聚合、排序或其他操作，以生成最终结果。

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

### 优化方案——数据倾斜 / join调优
1. 预处理数据：在spark之前处理数据。保证spark运行效率。
2. 过滤少数倾斜Key：某些key异常的数据较多。可以动态判定哪些key的数据量最多然后再进行过滤，那么可以使用sample算子对RDD进行采样，然后计算出每个key的数量，取数据量最多的key过滤掉即可。
3. 提高shuffle并行度：`spark.sql.shuffle.partitions`代表了shuffle read task的并行度，默认是200比较小。增加shuffle read task的数量，可以让原本分配给一个task的多个key分配给多个task，从而让每个task处理比原来更少的数据。（方案对于某个key倾斜不适用，因为这个key还是被打到了某一个task）
4. Join使用broadcast/reduce类使用随机前缀打散key：对于join改成broadcast实际上是避免了shuffle。reduce方法实际上是降低了某一个key的数据量。
5. 两个表都很大的join，可以打散+聚合。



## shuffle调优
大多数Spark作业的性能主要就是消耗在了shuffle环节，因为该环节包含了大量的磁盘IO、序列化、网络数据传输等操作。

有两种shuffle，1.2以后默认的是HashShuffleManager，HashShuffleManager有着一个非常严重的弊端，就是会产生大量的中间磁盘文件，进而由大量的磁盘IO操作影响了性能。后面变成了SortShuffleManager，这个对于shuffle有所改进，对于临时文件会最后合并为一个文件，因此每个task就只会有一个磁盘文件。

### HashShuffleManager
#### shuffleWrite/shuffleRead
**shuffle write**阶段，主要就是在一个stage结束计算之后，为了下一个stage可以执行shuffle类的算子（比如reduceByKey），而将每个task处理的数据按key进行“分类”。所谓“分类”，就是对相同的key执行hash算法，从而将相同key都写入同一个磁盘文件中，而每一个磁盘文件都只属于下游stage的一个task。在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去。

>那么每个执行shuffle write的task，要为下一个stage创建多少个磁盘文件呢？很简单，**下一个stage的task有多少个，当前stage的每个task就要创建多少份磁盘文件**。比如下一个stage总共有100个task，那么当前stage的每个task都要创建100份磁盘文件。如果当前stage有50个task，总共有10个Executor，每个Executor执行5个Task，那么每个Executor上总共就要创建500个磁盘文件，所有Executor上会创建5000个磁盘文件。由此可见，未经优化的shuffle write操作所产生的磁盘文件的数量是极其惊人的。

shuffle read，通常就是一个stage刚开始时要做的事情。此时该stage的每一个task就需要将上一个stage的计算结果中的所有相同key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行key的聚合或连接等操作。由于shuffle write的过程中，task给下游stage的每个task都创建了一个磁盘文件，因此shuffle read的过程中，**每个task只要从上游stage的所有task所在节点上，拉取属于自己的那一个磁盘文件即可**。

shuffle read的拉取过程是一边拉取一边进行聚合的。每个shuffle read task都会有一个自己的buffer缓冲，**每次都只能拉取与buffer缓冲相同大小的数据**，然后通过内存中的一个Map进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到buffer缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。

HashShuffleManager可以进行优化，实现复用磁盘文件。假设第一个stage有50个task，第二个stage有100个task，总共还是有10个Executor，每个Executor执行5个task。那么原本使用未经优化的HashShuffleManager时，每个Executor会产生500个磁盘文件，所有Executor会产生5000个磁盘文件的。但是此时经过优化之后，每个Executor创建的磁盘文件的数量的计算公式为：CPU core的数量（1） * 下一个stage的task数量。也就是说，每个Executor此时只会创建100个磁盘文件（下一个stage的task数量），所有Executor只会创建1000个磁盘文件。

## sortShuffleManager
SortShuffleManager的运行机制主要分成两种，一种是普通运行机制，另一种是bypass运行机制。当shuffle read task的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值时（默认为200），就会启用bypass机制。

>普通运行机制

在该模式下，数据会先写入一个内存数据结构中，此时根据不同的shuffle算子，可能选用不同的数据结构。如果是reduceByKey这种聚合类的shuffle算子，那么会选用Map数据结构，一边通过Map进行聚合，一边写入内存；如果是join这种普通的shuffle算子，那么会选用Array数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。

在溢写到磁盘文件之前，**会先根据key对内存数据结构中已有的数据进行排序**。排序过后，会分批将数据写入磁盘文件。默认的batch数量是10000条，也就是说，排序好的数据，会以每批1万条数据的形式分批写入磁盘文件。

一个task将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是merge过程，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中。此外，由于一个task就只对应一个磁盘文件，也就意味着该task为下游stage的task准备的数据都在这一个文件中，因此还会单独写一份索引文件，其中标识了下游各个task的数据在文件中的start offset与end offset。

SortShuffleManager由于有一个磁盘文件merge的过程，因此大大减少了文件数量。比如第一个stage有50个task，总共有10个Executor，每个Executor执行5个task，而第二个stage有100个task。由于每个task最终只有一个磁盘文件，因此此时每个Executor上只有5个磁盘文件(因为当前只有5个task执行)，所有Executor只有50个磁盘文件。

> bypass 

bypass运行机制的触发条件如下： 
* shuffle map task数量小于spark.shuffle.sort.bypassMergeThreshold参数的值。 
* 不是聚合类的shuffle算子（比如reduceByKey）。

此时task会为每个下游task都创建一个临时磁盘文件，并将数据按**将key写入对应的磁盘文件之中**。当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

该过程的磁盘写机制其实跟未经优化的HashShuffleManager是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。因此少量的最终磁盘文件，也让该机制相对未经优化的HashShuffleManager来说，shuffle read的性能会更好。

而该机制与普通SortShuffleManager运行机制的不同在于：第一，磁盘写机制不同；第二，不会进行排序。也就是说，启用该机制的最大好处在于，shuffle write过程中，不需要进行数据的排序操作，也就节省掉了这部分的性能开销。


相关参数：https://tech.meituan.com/2016/05/12/spark-tuning-pro.html
# 其他问题
## Spark程序执行，有时候默认为什么会产生很多task，怎么修改默认task执行个数？

- 有很多小文件的时候，有多少个输入block就会有多少个task启动
- spark中有partition的概念，每个partition都会对应一个task，task越多，在处理大规模数据的时候，就会越有效率
## 为什么shuffle性能差/shuffle发生了什么
1. 各个节点上的相同key都会先写入本地磁盘文件中，然后
2. 其他节点需要通过网络传输拉取各个节点上的磁盘文件中的相同key。
3. 相同key都拉取到同一个节点进行聚合操作时，还有可能会因为一个节点上处理的key过多，导致内存不够存放，进而溢写到磁盘文件中。
 
因此在shuffle过程中，可能会发生大量的**磁盘文件读写的IO操作**，以及**数据的网络传输操作**。磁盘IO和网络数据传输也是shuffle性能较差的主要原因。

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



