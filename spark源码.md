- [Spark执行环境](#spark执行环境)
- [RPC环境](#rpc环境)
  - [序列化管理器 SerializerManager](#序列化管理器-serializermanager)
  - [广播管理器 BroadcastManager](#广播管理器-broadcastmanager)
  - [map任务输出追踪器 MapOutputTrackerManager](#map任务输出追踪器-mapoutputtrackermanager)
  - [输出提交协调器 OutputCommitCoordinator](#输出提交协调器-outputcommitcoordinator)
- [存储体系](#存储体系)
  - [核心函数](#核心函数)
- [调度系统](#调度系统)
  - [RDD](#rdd)
  - [stage](#stage)
  - [Dag](#dag)
    - [DagScheduler的常用方法](#dagscheduler的常用方法)
    - [DagScheduler与Job提交](#dagscheduler与job提交)
  - [TaskScheduler](#taskscheduler)
  - [小结](#小结)

# Spark执行环境
# RPC环境
NettyRpcEnv需要一个消息调度器，dispatcher
![在这里插入图片描述](https://img-blog.csdnimg.cn/d09054571dc64be69850662833d886f9.png)
1. 序号1表示调用Inbox的post方法将消息放入messages列表
2. 序号2表示将消息的Inbox相关的endpointData放入receivers
3. 序号3表示messageLoop每次循环首先从receivers中获取EndpointData
4. 序号4表示endpointData中inbox的process方法对消息进行处理
## 序列化管理器 SerializerManager

## 广播管理器 BroadcastManager

## map任务输出追踪器 MapOutputTrackerManager

## 输出提交协调器 OutputCommitCoordinator

# 存储体系
![在这里插入图片描述](https://img-blog.csdnimg.cn/a03e94bab23b41a0b4fcd4d18679fb77.png)


存储体系从狭义上指 BlockManager，广义上包括了下面的所有。

- Block信息：id，storageLevel存储级别（use disk, use memory, use offheap, use deserialized, replication）, size

- BlockInfoManager: 管理block的锁资源，读锁是可重入共享锁，写锁是排他锁。 一个任务尝试执行线程时可以同时得到多个不同block的写锁和读锁，但是不能同时得到同一个block的读锁和写锁。读锁时可重入的，写锁不可重入。
- DiskBlockManager: 负责为逻辑Block与数据写入磁盘位置之间简历映射关系
- DiskStore：负责将Block存储到硬盘
- MemoryManager: spark除了把内存作为计算资源以外，还作为了存储资源。MemoryManager负责管理。spark把内存划分为了堆内存和堆外内存。堆内存时JVM堆内存的一部分，堆外内存时工作节点中系统内存的一部分空间。内存可以被分为StorageMemoryPool和ExecutionMemoryPool。

![在这里插入图片描述](https://img-blog.csdnimg.cn/8695657c5a82474ab14c31971805fd82.png)

其中有一个特殊的unifiedMemoryManager，可以把heap的内存作为一个软边界。即，计算资源或者存储资源可以借用。

- MemoryStore: 在内存之中执行存储。需要注意的是内存模型，对于MemoryManager模型中，堆内和堆外模型是透明无法区分的。两者的和是MaxMemory，memoryUsed可以分为blockMemoryUsed是存储了实体了，另外就是正在执行展开block的内容currentUnrollMemory。
- BlockManager：运行在每个节点上(Driver/executor)，提供堆本地或者远端节点上的内存，磁盘和堆外内存中Block的管理。其中在读取data的时候，如果可以写到内存，现在内存中写，如写满写disk。读取时候先哦那个memory中读，然后看disk中。
- BlockManagerMaster：对Executor和driver上的BlockManager进行管理。Driver的BlockManagerMaster会实例化一个endpoint，其他的所有BlockManagerMaster都会持有这个endpoint的RpcEndpointRef。

BlockManagerMaster负责发送消息，endpoint负责接收和处理，slaveEndpoint负责处理MasterEndpoint的命令。


## 核心函数

- MemoryStore
  - putBytes：整个写入，用字节存储，不需要展开
  - putIteratorAsValues: 把对象非序列化存储
    - putIterator：一条条写：不断的展开并申请内存，但不序列化。
  - putIteratorAsBytes：把对象序列化存储
    - putIterator：一条条写：不断的展开并申请内存，序列化。
- DiskStore
  - putByte：序列化存入磁盘
    - put: 检查是不是有了这个blockId，然后写
- BlockManager
  - putSingle:写入一个有单个对象组成的块, 调用doPutIterator
    - doPutIterator： 存储对象。优先使用memory，根据序列化选项调用MemoryStore不同的方法，如果存放不开再序列化放入disk；其次使用Disk序列化。分别调用memoryStore.putIteratorAsValues/putIteratorAsBytes, diskStore.put
  - doPutBytes：写入字节数组数据。分别调用memoryStore.putIteratorAsValues/putBytes, diskStore.putBytes



```cpp
if (level.useMemory) {
          // Put it in memory first, even if it also has useDisk set to true;
          // We will drop it to disk later if the memory store can't hold it.
          val putSucceeded = if (level.deserialized) {
            saveDeserializedValuesToMemoryStore(blockData().toInputStream()) // call memoryStore.putIteratorAsValues
          } else {
            saveSerializedValuesToMemoryStore(readToByteBuffer()) // call memoryStore.putBytes
          }
          if (!putSucceeded && level.useDisk) {
            logWarning(s"Persisting block $blockId to disk instead.")
            saveToDiskStore()
          }
        } else if (level.useDisk) {
          saveToDiskStore() // call diskStore.putBytes() 去执行写
        }
```
https://www.cnblogs.com/zhuge134/p/10995585.html

# 调度系统
1. 用户提交的job会被转换成一系列的RDD并通过RDD的依赖关系构建DAG，然后将RDD组成的DAG提交给调度系统
2. DAGScheduler负责接收RDD组成的DAG，将一系列RDD划分成不同的stage。根据stage不同的类型，给每个stage中未完成的partition创建不同的类型的task。每个stage有多少未完成的partition，就有多少task。每个stage的task会以taskset的形式，提交给taskscheduler。
3. TaskScheduler负责对taskset进行管理，并将TaskSetManager添加到调度池，然后将task的调度交给调度后端接口（SchedulerBackend）。SchedulerBackend申请taskScheduler，按照task调度算法对调度池中的所有TaskSetManager进行排序，然后对TaskSet按照最大本地性原则分配资源，并且在各个节点执行task
4. execute task：执行任务。

## RDD
包含的信息
1. deps：dependency序列
2. partitioner：当前RDD的分区计算器
3. id，name，storageLevel
4. partitions：存储当前RDD的所有扽去的数组
另外还有一个RDDInfo


接口方法：
1. compute：对RDD分区计算
2. getPartitions：获取当前RDD的所有分区。
3. getDependencies：获取当前RDD的所有依赖
4. getPerferredLocations：获取某一分区的偏好位置

模板方法：
1. partitions：默认实现：从checkPoint找，读取partitions，调用Getpartitons
2. perferredLocations：默认实现：从checkPoint保存的rdd中getPerferredLocations找，调用自身Getpartitons
3. dependencies：默认实现：从checkPoint中获取rdd并封装成OneToOneDependency，调用自身getDependencies

## stage
1. id,rdd(当前stage包含的rdd)
2. numTasks：当前stage的task数量
3. parents：当提前stage的父stage list
4. firstJobId：第一个提交当前stage的jobid。
5. numPartitions：当前stage的分区数量。实际为rdd的分区数量。
6. jobIds：当前stage所属的Job的身份表示集合。一个stage可能属于多个job。

子RDD可以有多个上游RDD，但是partition必须11对应
- 窄依赖：OneToOneDependency和RangeDependency
- shuffle依赖：核心是分区计算器Partitioner


ShuffleMapStage：
1. numAvailableOutputs：可用的map任务的输出数量，代表执行成功的map数量。最终应等于numPartitions
2. outputLocs：各个map任务对应的mapStatus列表的映射关系。

## Dag
1. 所有组件通过投递DAGSchedulerEvent来使用DAGScheduler
2. DAGScheduler通过DAGSchedulerProcessLoop处理event
3. JobListener对每个task监听，Job执行成功/失败会调用taskSucceed/JobFailed
4. JobWaiter实现了JobListener,最后确定Job的状态

RDD在进行各种转换的时候已经保存了自己的血缘关系。也就是说，尽管我们submitJob时候提交的信息是最后一个rdd，但是我们可以根据RDD推出来dependency。

### DagScheduler的常用方法
1. getCacheLocs: 如果当前的cacheLocs没有RDD对应的位置信息，且RDD的存储级别不是NONE。构造RDD各个分区的RDDBlockId数组，然后调用BlockManagerMaster.getLocations(blockId)获取每个RDDBlockedId储存的位置信息序列，封装为TaskLocation。然后返回本地的cacheLocs中。
2. getPerferredLocsInternal: 获得RDD的指定分区的偏好位置。有visited map防止对RDD分区的重复访问。 
   1. 调用getCacheLocs返回RDDLocation，如果能得到就返回位置。
   2. 否则看看perferredLocation有没有位置偏好信息，如果有封装成TaskLocation返回。
   3. 否则遍历rdd的窄依赖，获取窄依赖中RDD的同一分区的偏好位置，找到一个就可以返回了。

### DagScheduler与Job提交

核心：**反向驱动stage构建，正向提交stage**

- runJob： 提交入口。调用`submitJob`提交job，返回JobWaiter对象。JobWaiter等待Job处理完毕。
- submitJob: 根据调用RDD的partition数量得到当前Job的最大分区数。如果Job分区数量大于0，向eventProcessActor发送JobSubmitted。

> 处理job提交
- handleJobSubmitted：DAGScheduler实际执行的方法。首先调用`createResultStage`创建出来最后的ResultStage(因为我们实际执行的一定是最后一个rdd去调用runJob) 。更新各种量，向LiveListenerBus post消息。调用`submitStage`方法提交finalStage。
  
> 构建stage

在遍历rdd依赖的过程中按深度优先遍历，每遇到一个shuffle依赖就创建一个stage，所有上游的stage创建完成后，最后再创建一个ResultStage。

- createResultStage: 根据当前的rdd和jobId，调用`getOrCreateParentStage`方法得到所有的父stage list，然后new 构建ResultStage。
  - getOrCreateParentStage：获取父Stage列表。调用`getShuffleDependencies`得到据rdd的所有ShuffleDependency。调用`getOrCreateShuffleMapStage`每一个dep都创建一个stage。
    - getShuffleDependencies： 根据RDD，bfs的得到所有的shuffle依赖。
    - getOrCreateShuffleMapStage： 根据shuffleDependency，得到shuffleId，如果从shuffleIdToMapStage里找不到，说明还没创建。就调用`getMissAcestorShuffleDependencies`方法，得到还没创建shuffleMapStage的祖先的shuffleDependency。最后调用`createShuffleMapStage`为当前shuffleDependency创建一个mapStage.
      - getMissAcestorShuffleDependencies: 得到祖先的shuffleDependency
      - createShuffleMapStage: 从shuffleDependency得到RDD，task数量是rdd的partition数量。递归调用`getOrCreateParentStage`得到父stage，创建出来当前的stage。 更新各种映射关系。调用mapOutputTracker.containsShuffle。看看是不是有当前shuffleId对应的mapStatus。如果没有，在MapOutputTrackerMaster上注册shuffleId和对应的MapStatus的映射关系。如果有就根MapOutputTrackerMaster据缓存的MapStatus，更新当前stage的outputLocos信息。

因为stage可以尝试，所以当前stage可能已经执行过。在上一次的执行中，部分map任务可能执行成功了并且MapOutputTrackerMaster缓存了MapStatus，因此只需要复制过来，避免重复计算。

> 提交ResultStage
- submitStage： 根据stage得到当前的jobId并检查合法。如果当前stage还没提交（waitingStage，runningStage，failedStage都不含）执行以下操作。调用`getMissingParentStages`方法，得到当前stage的所有未提交的父stage。如果不存在未提交的父stage，调用`submitMissingTasks`方法，提交当前stage所有未提交的task。如果存在未提交的父stage，依次调用`submitStage`提交父stage，并将当前stage加入waitingStage，等待父依赖完成。
  - getMissingParentStage：当前stage的rdd分区中存在没有对应TaskLocation序列的分区。说明stage的某个上由ShuffleMapStage的某个分区任务没有执行完成。然后根据依赖得到上游的某个stage发现还不可用，说明还没开始执行。

> 提交还未计算的task

- submitMissingTasks: 
  1. 清空当前stage的pendingPartitions
  2. 调用stage的`findMissingPartitions`找到stage中所有分区没有完成计算的分区索引
  3. 将当前stage加入runningStages
  4. 调用OutputCommitCoordinator启动对当前stage输出提交的HDFS协调
  5. 调用DAGScheduler的`getPerferredLocs`获取partitionToCompute的每一个分区的偏好位置。
  6. 调用stage的`makNewStageAttemp`方法执行尝试，并且向listenerBus投递stageSubmit事件。
  7. 执行一些checkpoint操作，进行序列化并且广播序列化对象。shuffle序列化当前stage的rdd和shuffleDep；Result序列化rdd和计算func。
  8. 如果是shuffleMapStage，为每一个分区创建一个ShuffleMapTask。如果是ResultStage，为每一个分区创建一个ResultTask。
  9. 如果有task需要执行，将task处理的分区索引添加到stage的pendingPartitions中，然后对这批task创建一个taskSet，调用TaskScheduler的submitTasks提交。
  10. 如果没有task执行，把stage标记为完成，并且提交stage的子stage。

> Task结果的处理

taskEnded方法去处理Task的结果。
- ShuffleMapTask:所有的状态信息MapStatus都追加到ShuffleMapStage的outputLocs缓存中。  
  - 如果所有分区的task都成功了，注册outputLoc的MapStatus到MapOutputTrackerMaster的mapStatus里，以方便下游stage的task去读取输入数据所在的位置信息。
  - 如果某个Task失败了，重新提交stage
  - 都成功以后，唤醒下游stage。
- ResultTask：所有的成功ResultTask成功以后，表即ResultStage成功并通知JobWaiter对各个task结果去收集处理。

具体的在`handlerTaskCompletion`：
- ResultTask: 成功以后在numFinished+1，如果numFinished === numPartitions表示全部分区的task都完成了，把当前stage标为完成。向LiveListenerBus post jobEnd事件。调用JobWaiter的taskSucceeded方法来处理job每个task的执行结果。
- ShuffleMapTask：task成功以后再pendingPartition减去当前的taskPartitionId，如果当前stage的pendingPartition是空的，表示所有task都完成了。把当前的MapStatus信息，更新到MapOutputTrackerMaster里，并且标记当前stage完成。检查stage的isAvailable（outputNumber和numPartition是否相等）。如果有任务失败了，重新提交当前stage；否则把当前stage的所有job都标记成功。然后调用`submitWaitingChildStage`提交子stage。

## TaskScheduler
依赖于 TaskSetManager，TaskScheduler， SchedulerBackend，调度池和调度算法

TaskSchedulerImpl接受dag给每个stage创建的taskset，然后按照调度算法将资源分配给task，再把task交给spark集群的不同节点运行。其中考虑了数据计算本地性。

TaskSchedulerImpl初始化的时候创建了pool并且传入了对应的调度算法。TaskSchedulerImpl启动的时候同步启动schedulerBackend。

- `submitTasks`: submitTasks是taskScheduler的入口。首先创建TaskSetManager并将manager添加到pool中，调度池决定了如果存在多个TaskSet在排队应该如何进行排序。调用SchedulerBackEnd.reviveOffer()给task分配资源并运行。
- `reviveOffers`：通过rpc模块给DriverEndPoint发送一个消息，DriverEndPoint调用`makeOffers`方法。把executor计算资源封装成为workOffer资源对象，调用executor的`launchTask`。

> 资源分配： TaskSchedulerImpl根据TaskSet优先级（调度池），黑名单，本地性等因素给出要实际运行的任务。我们使用round-robin的方式将任务分配到各个executor上，以使得计算资源的 使用更均衡。`resourceOffers`这个方法由调度后端调用，调度后端会将可用的executor资源告诉TaskSchedulerImpl。
- `resourceOffers`: 遍历workerOffer序列，更新executor，host，rack的关系。随机打乱worker，避免任务总是分给同一组worker。根据每个workerOffer的core数量决定可分配任务数量。调用rootPool的`getSortedTaskSetQueue`方法对pool中的taskSetManager按照调度算法排序。遍历排序后的TaskSetManager，按照最大本地性原则，调用`resourceOfferSingleTaskSet`方法，给单个TaskSet中的task提供资源。返回TaskDescription列表，表示已经获得资源的任务列表。
  - `resourceOfferSingleTaskSet`: 根据不同的workerOffer和task的本地性情况，分配资源。轮询每一个executor，分别为其分配一个Task。检查这个executor上的cpu资源是否够用。根据最大允许的本地性级别取出能够在这个executor上执行的任务（调用`TaskSetManager.resourceOffer()`）。 将这个任务加入传进来的集合中。
  - `TaskSetManager.resourceOffer()`： 根据本地等待事件重新计算本地性级别。对task进行序列化，如果序列化后大于128MB会报错。生成真正的TaskDescription给上层调用。
  
> 执行task
- `DriverEndpoint.launchTasks()`：得到了TaskDescription，对每一个task满足RPC大小的前提下，通过RPC发送到对应的executor上执行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b3a88df80a1c48f18cf6f32568e21d98.png)

关于本地性有五个级别PROCESS_LOCAL(本地进程), NODE_LOCAL（本地节点）, NO_PREF（没有偏好）, RACK_LOCAL（本地机架）, ANY（任何），每个本地性级别会进行多轮分配， 每一轮依次轮询每个executor，为每一个executor分配一个符合本地性要求的任务。这样一轮下来每个executor都会分配到一个任务，显然大多数情况下，executor的资源是不会被占满的。

## 小结
任务在driver中从诞生到最终发送的过程：
- DAGScheduler对作业计算链按照shuffle依赖划分多个stage，提交一个stage根据个stage的一些信息创建多个Task，包括ShuffleMapTask和ResultTask, 并封装成一个任务集（TaskSet）,把这个任务集交给TaskScheduler。
- TaskSchedulerImpl将接收到的任务集加入调度池中，然后通知调度后端SchedulerBackend。
- CoarseGrainedSchedulerBackend收到新任务提交的通知后，检查下现在可用 executor有哪些，并把这些可用的executor交给TaskSchedulerImpl。
- TaskSchedulerImpl根据获取到的计算资源，根据任务本地性级别的要求以及考虑到黑名单因素，按照round-robin的方式对可用的executor进行轮询分配任务，经过多个本地性级别分配，多轮分配后最终得出任务与executor之间的分配关系，并封装成TaskDescription形式返回给SchedulerBackend。
- SchedulerBackend拿到这些分配关系后，就知道哪些任务该发往哪个executor了，通过调用rpc接口将任务通过网络发送即可。