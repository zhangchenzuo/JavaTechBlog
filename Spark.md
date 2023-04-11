# spark 框架和运行原理
## Spark组件
- Spark Core： 
将 分布式数据 抽象为弹性分布式数据集（RDD），实现了应用**任务调度**、RPC、序列化和压缩，并为运行在其上的上层组件提供API。它提供了**内存计算的能力**。

- Spark SQL：
Spark Sql 是Spark来**操作结构化数据的程序包**，可以使用SQL语句的方式来查询数据。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。

## 基本框架
https://www.hadoopdoc.com/spark/spark-principle

<!-- - Application：
用户编写的Spark应用程序，包含了driver程序以及在集群上运行的程序代码。 -->

- Cluster Manager：控制整个集群，监控worker。在standalone模式中即为Master主节点，控制整个集群，监控worker。在YARN模式中为资源管理器

- Worker：
从节点，负责控制计算节点，启动Executor或者Driver。集群中任何一个可以运行spark应用代码的节点。**Worker就是物理节点**，可以在上面**启动Executor/worker进程**。一个worker上的memory、cpu由多个executor共同分摊。

  - Driver：
  Spark中的Driver即运行Application的main函数并**创建SparkContext**，创建SparkContext的目的是为了**准备Spark应用程序的运行环境**，在Spark中由SparkContext负责与Cluster Manager通信，**进行资源申请、任务的分配和监控**等，当Executor部分运行完毕后，Driver同时负责**将SparkContext关闭**。

  - Executor：
  在每个Worker上为某应用启动的一个**进程**，该进程**负责运行Task，并且负责将数据存在内存或者磁盘上**，每个任务都有各自独立的Executor。Executor是一个执行Task的容器。它的主要职责是：
    - 初始化程序要执行的上下文SparkEnv，解决应用程序需要运行时的jar包的依赖，加载类。
    - 向cluster manager汇报当前的任务状态。
  Executor是一个应用程序运行的监控和执行容器。
    - 心跳的方法和diver联系。

    executor会存在于整个application生命周期。task执行完之后executor就会把结果发送给驱动程序。如果application代码里调用RDD的缓存函数，如cache()或者persist()，executor将会通过Block Manager给Spark RDD提供缓存机制。

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

# Spark运行基本流程
1. 为应用构建起基本的运行环境，即由Driver创建一个SparkContext进行资源的申请、任务的分配和监控。
2. 资源管理器为Executor分配资源，并启动Executor进程
3. SparkContext根据RDD的依赖关系构建DAG图，DAG图提交给DAGScheduler解析成Stage，然后把一个个TaskSet提交给底层调度器TaskScheduler处理。
4. Executor向SparkContext申请Task，TaskScheduler将Task发放给Executor运行并提供应用程序代码。
5. Task在Executor上运行把执行结果反馈给TaskScheduler，然后反馈给DAGScheduler，运行完毕后写入数据并释放所有资源。

# RDD
- DAG：有向无环图，反应了RDD之间的依赖关系。

# 与MR对比
## 优点
Spark速度更快，因为可以使用多线程并且可以使用内存加速计算。