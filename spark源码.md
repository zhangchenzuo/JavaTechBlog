- [Spark执行环境](#spark执行环境)
- [RPC环境](#rpc环境)
  - [序列化管理器 SerializerManager](#序列化管理器-serializermanager)
  - [广播管理器 BroadcastManager](#广播管理器-broadcastmanager)
  - [map任务输出追踪器 MapOutputTrackerManager](#map任务输出追踪器-mapoutputtrackermanager)
  - [输出提交协调器 OutputCommitCoordinator](#输出提交协调器-outputcommitcoordinator)
- [存储体系](#存储体系)
  - [核心函数](#核心函数)

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