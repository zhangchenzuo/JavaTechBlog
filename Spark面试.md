- [spark 框架和运行原理](#spark-框架和运行原理)
  - [Spark组件](#spark组件)
  - [运行框架](#运行框架)
  - [集群运行](#集群运行)
- [Job，Stage，Task](#jobstagetask)
  - [Job](#job)
  - [stage](#stage)
  - [宽窄依赖](#宽窄依赖)
  - [Task](#task)
- [spark的数据结构](#spark的数据结构)
  - [RDD](#rdd)
    - [RDD持久化](#rdd持久化)
  - [DataFrame](#dataframe)
  - [DataSet](#dataset)
  - [三种数据结构的对比](#三种数据结构的对比)
  - [分布共享变量](#分布共享变量)
  - [累加器 只写](#累加器-只写)
  - [广播变量 只读](#广播变量-只读)
- [与MR对比](#与mr对比)
  - [优点](#优点)

# spark 框架和运行原理

## Spark组件
- Spark Core： 
将 分布式数据 抽象为弹性分布式数据集（RDD），实现了应用**任务调度**、RPC、序列化和压缩，并为运行在其上的上层组件提供API。它提供了**内存计算的能力**。

- Spark SQL：
Spark Sql 是Spark来**操作结构化数据的程序包**，可以使用SQL语句的方式来查询数据。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。

## 运行框架

master/worker指的是资源的调度层面，申请内存等。driver/executor则是在任务的执行调度层面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d5f1c3b621c4e3a80af25c4e771f01f.png)
- Cluster Manager（master）：控制整个集群，监控worker。在standalone模式中即为Master主节点，控制整个集群，监控worker。在YARN模式中为资源管理器

- Worker：
从节点，负责控制计算节点，启动Executor或者Driver。集群中任何一个可以运行spark应用代码的节点。**Worker就是物理节点**，可以在上面**启动Executor/worker进程**。一个worker上的memory、cpu由多个executor共同分摊。

  - Driver：
  运行Application的main函数并**创建SparkContext**，**准备Spark应用程序的运行环境**。在Spark中由SparkContext负责与Cluster Manager通信，**进行资源申请、任务的分配和监控**等，当Executor部分运行完毕后，Driver同时负责**将SparkContext关闭**。
    1. 将用户程序转化为任务：程序从输入数据创建一些系列RDD，使用转化操作派生出新的RDD，最后使用行动操作收集或者存储结果RDD数据。实际上是隐式的创建了一个由操作组成的逻辑上的**有向无环图（Directed Acyclic Graph， DAG）**。当驱动器运行时，会转化为物理执行计划。
    2. 为执行器节点调度任务：执行器启动后会向驱动器注册自己，每个执行器节点都代表一个能够执行任务和存储RDD数据的进程。执行器会把任务基于数据所在位置分配给合适的执行器进程。任务执行时，执行器会缓存这些数据，驱动器会跟踪这些数据的缓存位置，并利用这些信息来调度以后的任务，以减少网络传输。


  - Executor：
  在每个Worker上为某应用启动的一个**JVM进程**，该进程**负责运行Task，并且负责将数据存在内存或者磁盘上**，每个任务都有各自独立的Executor。Executor是一个执行Task的容器。任务相互独立，即使其他的执行器节点有崩溃也不会影响自身。它的主要职责是：
    1. 负责执行组成spark应用的任务，并将结果返回给驱动器进程
    2. 通过自身的Block Manager，为用户进程中要求缓存RDD提供内存式存储，RDD直接缓存在执行器内存中。

- 每个Application都有自己专属的Executor进程，并且该进程在Application运行期间一直驻留。**Executor进程以多线程的方式运行Task**。
- Spark运行过程与资源管理器无关，只要能够获取Executor进程并保存通信即可。
## 集群运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/bef07a0b0a0d4df0a65861d70566385e.png)

1. Driver进程启动，创建一个SparkContext进行资源的申请、任务的分配和监控。发送请求到Master节点上,进行Spark应用程序的注册。
2. SparkContext向资源管理器（Standalone，Mesos，Yarn）申请运行Executor资源。Master在接受到Spark应用程序的注册申请之后,会发送给Worker进行资源的调度和分配。Worker 在接受Master的请求之后，启动Executor来分配资源。Executor启动分配资源好后，向Driver进行反注册。
3. SparkContext根据RDD的依赖关系构建DAG图，DAG图提交给DAGScheduler解析成Stage，然后把一个个TaskSet提交给底层调度器TaskScheduler处理。 
4. Executor向SparkContext申请Task，TaskScheduler将Task发放给Executor运行并提供应用程序代码。
5. Task在Executor上运行把执行结果反馈给TaskScheduler，然后反馈给DAGScheduler，运行完毕后写入数据并释放所有资源。
   
# Job，Stage，Task
## Job
一个Job包含多个RDD及作用于相应RDD上的各种操作，它包含很多task的并行计算。它通常由一个或多个RDD转换操作和行动操作组成，这些操作会被划分为一些Stage
## stage
是Job的基本调度单位，由一组共享相同的Shuffle依赖关系的任务组成。有一个job分割多个stage的的点在于shuffle，宽依赖Stage需要进行Shuffle操作，而窄依赖Stage则不需要。
## 宽窄依赖
- 窄依赖：
指**父RDD的每一个分区最多被一个子RDD的分区所用**，表现为一个父RDD的分区对应于一个子RDD的分区，和两个父RDD的分区对应于一个子RDD 的分区。

- 宽依赖：
**指子RDD的分区依赖于父RDD的所有分区，这是因为shuffle类操作**。

> stage的分类
在Spark中，Stage可以分成两种类型。

- ShuffleMapStage：
  - 这种Stage是以Shuffle为输出边界
  - 其输入边界可以是从外部获取数据，也可以是另一个ShuffleMapStage的输出
  - 其输出可以是另一个Stage的开始
  - ShuffleMapStage的最后Task就是ShuffleMapTask
  - 在一个Job里可能有该类型的Stage，也可以能没有该类型Stage
- ResultStage
  - 这种Stage是直接输出结果
  - 其输入边界可以是从外部获取数据，也可以是另一个ShuffleMapStage的输出
  - ResultStage的最后Task就是ResultTask
  - 在一个Job里必定有该类型Stage
## Task
被发送到executor上的工作单元。每个Task负责计算一个分区的数据。可以分为需要shuffle的shuffleMapTask和resultTask。


一个Application由一个Driver和若干个Job构成，一个Job由多个Stage构成，一个Stage由多个没有Shuffle关系的Task组成。

用户提交的Job会提交给DAGScheduler，Job会被分解成Stage，Stage会被细化成Task，Task简单的说就是在一个数据partition上的单个数据处理流程。

当执行一个Application时，Driver会向集群管理器申请资源，启动Executor，并向Executor发送应用程序代码和文件，然后在Executor上执行Task，运行结束后，执行结果会返回给Driver，或者写到HDFS或者其它数据库中




需要注意的是，sprak都是**惰性计算**，只用在第一次的一个行动操作中用到时，才会真正计算。也就是说spark其实是了解了整个转化链后，只有真正执行才会计算。

# spark的数据结构
## RDD
弹性分布式数据集（Resilient Distributed Dataset， RDD）。是一个不可变（只读）的分布式对象集合，每个RDD可以有多个分区。它是spark的基础数据结构，具有内存计算能力、数据容错性以及数据不可修改特性。
### RDD持久化
让spark持久化存储一个RDD，计算出的RDD节点会分别保存它们所求出的分区数据。如果一个有持久化数据的节点发生故障，spark会在需要缓存数据时候重算。默认情况下，`persist()`会把数据以**序列化的形式缓存在JVM的堆空间**中。也可以通过改变参数，存在硬盘上。

- DAG：有向无环图，反应了RDD之间的依赖关系。
## DataFrame
Dataframe也是一种不可修改的分布式数据集合，它可以按列查询数据，可以当成数据库里面的表看待。可以对**数据指定数据模式**（schema）。也就是说DataFrame是Dataset[row]的一种形式
## DataSet
Dataset是DataFrame的扩展，它提供了类型安全，面向对象的编程接口。知道每个row的字段类型。


## 三种数据结构的对比
![在这里插入图片描述](https://img-blog.csdnimg.cn/41f79a3fb1a246428e68e9c78181fb90.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAemN6NTU2NjcxOQ==,size_18,color_FFFFFF,t_70,g_se,x_16)

- RDD类型，只是包含了对应的类型参数，但是并不清楚Person类的内部结构。而DataFrame可以得到Column的名字。
- RDD是分布式的 Java对象的集合。DataFrame是分布式的Row对象的集合。
- Dataset可以认为是DataFrame的一个特例，主要区别是Dataset每一个record存储的是一个强类型值而不是一个Row。

## 分布共享变量

两种类型的共享变量：累加器（accumulator）和广播变量（broadcast variable）；累加器用于对信息进行聚合，广播变量用于高效分发较大的变量。

## 累加器 只写
在向spark传递函数时候，可以使用驱动器程序中的变量，集群中运行的每个任务都会得到这些任务的一个新的副本，更新这些副本的值并不会影响驱动器中的值。

而累加器是一个共享变量，将工作节点中的值聚合到驱动器程序中。比如可以用于统计节点中出现错误的行数。

## 广播变量 只读
广播变量可以高效的向所有工作节点发送一个较大的只读值或者表。

spark会自动将闭包中的变量都复制并发送到工作节点上，这样会低效。其实多个工作节点可以共用同一个表变量。
# 与MR对比
## 优点
Spark速度更快，因为可以使用多线程并且可以使用内存加速计算。