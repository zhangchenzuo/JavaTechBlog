- [中文](#中文)
  - [spark小文件问题](#spark小文件问题)
    - [spark小文件合并基本原理](#spark小文件合并基本原理)
    - [FileOutputCommitter V1](#fileoutputcommitter-v1)
    - [FileOutputCommitter V2文件提交机制](#fileoutputcommitter-v2文件提交机制)
    - [收益](#收益)
- [英文](#英文)
- [PII](#pii)
# 中文
表有多大: 
一小时有 5000万条beacon，每条大概几十K，一小时2，3个T
一天 20T。

我在的团队是hulu广告智能团队下的data team，我们主要做的内容是负责hulu和disney的广告系统的数据仓库的搭建。21年底我们团队开始做一个核心项目就是给disney+平台上一套广告系统，之前disney是没有广告系统的。因此我们搭建一个数仓平台给广告业务提供支持。

整个广告的data处理过程可以分为 实时流式处理 和 离线批处理。实时流式处理的团队利用kafaka或者aws上knesis接受广告埋点数据，dump数据并且根据业务需求做一些实时处理。 我所在的 离线批处理团队 拿到实时团队 dump的广告埋点数据，对这些数据进行批量处理。 

我们的输入数据会包括从client端拿到的用户行为数据，比如用户观看行为，投放的广告有25%，50%还是75%被观看了，以及从sever端拿到的广告被填充的具体信息，广告的meta id等等。以及很多在聚合阶段需要使用到的meta信息。 

我们输出的数据是大宽表的形式被多个下游团队使用，比如BI 商业分析团队，广告归因团队，和算法团队。

具体的工作内容在业务层面上： 我们会对原始收集到的json数据进行 解析，归一化，校验。然后使用client端和sever端的事实数据 加上 很多的广告meta信息，用户meta信息进行聚合。我们最后得到的数据表主要是3个大的主题上，一个revenue主题的数据，我们会在单个广告的维度上进行聚合得到 ads delivery数据，也就是看看投放的广告被用户观看的记录。这些数据在经过进一步的 反欺诈反作弊的MRC以后，会被过滤掉一些异常流量，比如一个广告是否在在短时间被多次观看，是不是有一些非法的ip观看了广告之类的。  还有就是supply主题的数据，我们会统计在一个pod上被选取的广告的情况，这些信息会帮助广告选取和投放团队进行优化。 最后就是 auction biding竞价的层面，也是给到算法或者BI团队看看是否最大化了投放收益。


在技术层面上：对于数仓ELT的开发方面，我们组主要做的是离线处理业务，所以大数据处理框架选择是spark框架，底层的存储部署之类的技术是在AWS上搭建，比如数据存储实际上使用的是aws的s3存储，任务调度使用开源框架airflow去调度，数据监控用的grafana等。这部分我的工作就主要是实现之前的数据清洗，数据聚合，和异常流量过滤的代码。以及使用airflow框架搭建pipeline去调度任务，配合qa测试以及对spark job的性能进行调优。包括对pipeline配置一些警报之类的。以及需要做一些spark job的性能调优之类的工作。


我是22年初加入的团队，正好那时候团队正在澄清项目需求和做流程设计。所以整个项目的发展过程我是从0.2，0.3的阶段开始加入到今年年出项目的上线，参与包括和上下游确定模型字段，表结构。以及我们内部需要确定对于数据处理的 具体 操作包括什么，比如字段归一化，对一些内容进行校验等。以及组内的大部分开发工作。 在这阶段我参与做了一些关于数据处理方案的设计工作。


对于spark的性能调优主要是关注job是不是可以正常运行，以及运行的时间是不是过长，资源分配是不是合理的。

这部分工作中，做spark性能调优是比较有挑战的问题，我们有一个job是负责处理流量过滤的，我们对于流量过滤的规则有小时级别的，有天级别的，还有一个最长的是取过去13天的数据进行聚合过滤。13天的数据大概会有几百T，因此executor就会出现有些task持续OOM失败，导致job无法被正常的执行。解决这类spark性能优化问题就是看具体的spark执行失败的原因，我们当时发现的情况是有某一个stage执行的shuffle特别大，执行时间比其他的要长接近1000倍。定位到代码上的原因是我们在做一个过滤的时候，对于IP有ipv4和ipv6两个黑名单，只有有一个被命中了就会被过滤。因此我们比较自然的做了一个orJoinCondition的操作。这个操作其实不太好，这种or的join条件会导致笛卡尔积的产生（指定不等值连接，不指定on）。因此导致大量的读写。
```sql
-- https://zhuanlan.zhihu.com/p/536880775
select * 
from [TABLE A] t1 
join [TABLE B] t2 
on t1.id = t2.id 
or t1.name = t2.name;
```
而且导致这个问题更加恶化的是，我们对于这个join table mrc_blocked_ip，我们有一个去重的操作，dropDuplicate。这个操作会把小表切成200个分区去执行，进一步导致了笛卡尔积的增加。变成了200*200 = 40000个task去执行。

解决的方法，首先修复一下or的逻辑，做一下判断，有ipv4和ipv6的分别判断。然后对于shuffle问题，我们直接直接broadcast出去。

还有一些可能发现有重复计算的rdd存在，这些rdd就可以cache或者persist一下，保证对一个RDD执行多次算子操作时，这个RDD本身仅仅被计算一次。

除了代码层面，还进行一些资源调优。包括增加executor-memory32G和executor number 64。类似的。

其余有挑战的问题：解决s3的读写性能问题，通过自己管理meta data，实现数据高性能写。解决空分区问题，当数据是空的时候需要查询到对应表的glue catalog信息，结合当前job的partition和location信息，自己生成catalog。


另外一个方面的工作就是，项目后期，有关于用户隐私法案的一些问题。美国那边的对数据隐私安全问题提出了一些chalange。这部分的工作是我和PM直接对接做design的。主要的需求是对于一些PII数据，我们不可以明文存储，并且需要应对RTD，用户有权要求删除。以及对于13岁以下的用户数据的PII不可以存储。其他用户的数据有一个最大存储期限，需要定期删除。

这部分工作也是需要结合一些广告业务的需求，因为我们其实会使用到一些用户的个人数据对用户的行为进行分析。比如用户画像之类的。用户的信息包括device-ad-id，ip address，accout-id这些。 对于我们来说有些数据是可以直接hash，我们也能使用的，比如ip，device id，因为我们主要是用于和黑名单上的ip和device进行比对进而进行流量过滤。但是ip信息对于有些下游团队是需要的，比如attribute团队做广告归因的时候就需要ip明文。



方案：
1. 对部分内容进行hash，比如device，ip等
2. 有一个compliance manager，可以按照用户的accout_id生成密钥，我们在进行数据处理时候，对于所有用户都请求密钥，然后对，ip等可能还需要解密的信息
等信息进行加密。
1. 对于可能的RTD情况，我们不在我们的数据里进行删除，而是删除compliance manager的数据。这样可以保证我们这里的数据在计算账单等信息的时候不出现不一致问题。
2. 对于U13内容，我们做了一个pipeline对这些字段进行删除，并且搬运到对外可见的db中。
3. 对于超过180天的数据定期物理删除


其他：

% action：Spark 调优（传统调优，数据偏斜，配置对不对），跟团队解决云上spark低效问题 (s3) 
    % 数据质量 （DQ），进行异步检查，发出对应的报警。DQ指标：是不是空，是不是变化的不正常，是不是一些信息的统计不正常，一个数据表有几十个。 异步的不会阻塞数据，如果出现问题，再重跑。 对于数据管理，正在做。

    ```
"account_id", "identity_id", "device_id", "device_ad_id_encrypted",
                            "ip_v4_encrypted", "ip_v4_hashed", "ip_v6_encrypted", "ip_v6_hashed"
```
## spark小文件问题
背景：大数据使用场景中，随着数据量持续增大，特别是Spark的使用加剧了小文件的产生，操作涉及的文件数达到了百万级，对作业的执行时间和性能都造成了显著影响

大数据领域的小文件，通常是指文件大小显著小于HDFS Block块（64MB或128MB）的文件，小文件过多会给HDFS带来严重的性能瓶颈（主要表现在NameNode节点元数据的管理以及客户端的元数据访问请求），并且对用户作业的稳定性和集群数据的维护也会带来很大的挑战。

Spark程序产生的文件数量直接取决于RDD中partition分区的数量和表分区的数量，需要注意的是，这里提到的两个分区，概念并不相同，**RDD的partition分区与任务并行度高度相关**，而表分区则是指的Hive表分区（对于Hive，**一个分区对应一个目录**，主要用于数据的归类存储），产生的文件数目一般是**RDD分区数和表分区数的乘积**。因此，当Spark任务并行度过高或表分区数目过大时，非常容易产生大量的小文件。

### spark小文件合并基本原理
Spark任务在执行过程中，通常都要经历从数据源获取数据，然后在内存中进行计算，最后将数据进行持久化的过程，其中有两个非常关键的操作：
1. executor端的task任务执行commitTask方法，将数据文件从task临时目录转移到Job临时目录；
2. driver端执行commitJob方法，将各个task任务提交的数据文件，从Job临时目录转移到Job的最终目标目录。

Spark小文件合并的基本原理：在executor端，各个task任务执行完commitTask方法提交数据后，先获取作业对应的所有小文件，然后按照**分区对小文件进行分组合并**，最后driver端执行commitJob方法，将合并后的数据文件转移到Job的最终目标目录。在Spark作业中，引入小文件合并功能的执行流程，如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a1864b9a0aab4807a57e3c7a475cbd39.png)


>spark写文件的方法：
通常情况下，在单机上写文件时，都会生成一个指定文件名的文件，而调用Spark DataFrame的writer接口来写文件时，会在指定路径下写入了多个数据文件。文件数量等于RDD的partition数量。是因为RDD中的数据是分散在多个Partition中的，而每个Partition对应一个task来执行，这些task会根据cores数量来并行执行。 这样做是为了解决 原子性和数据一致性 问题。（由于是多个task并行写文件，需要保证所有task写的所有文件要么同时对外可见，要么同时不可见。3个task的写入速度是不同的，那就会导致不同时刻看到的文件个数是不一样的。此外，如果有一个task执行失败了，会导致有2个文件残留在这个路径下）
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ff6bf1ffb7341d593b2ca9b555db7a7.png)

Spark有两种不同的文件提交机制，即FileOutputCommitter V1和FileOutputCommitter V2。
### FileOutputCommitter V1
FileOutputCommitter V1文件提交机制的基本工作原理，需要经历两次rename过程。
- 每个**task**先将数据写入到如下临时目录：`finalTargetDir/_temporary/appAttemptDir/_temporary/taskAttemptDir/dataFile`
- 等到task完成数据写入后，执行commitTask方法做第一次rename，将数据文件从**task临时目录**转移到如下临时目录：`finalTargetDir/_temporary/appAttemptDir/taskDir/dataFile`
- 最后，当所有task都执行完**commitTask**方法后，由Driver负责执行commitJob方法做第二次rename，依次将数据文件从job临时目录下的各个task目录中，转移到如下最终目标目录中，并生成_SUCCESS标识文件：`finalTargetDir/dataFile`

FileOutputCommitter V1文件提交机制的执行流程，如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/c91fb4a3d68b4639bff27cc9a38dc779.png)

优点是解决了数据一致性问题，但是因为有两次rename，效率很低。（s3是对象存储，仅仅通过数据唯一地址寻找数据，因此不支持rename操作，实际上是copy+delete）
### FileOutputCommitter V2文件提交机制
为了解决V1的性能问题，牺牲了一部分数据一致性问题。在方案上去掉了在commitJob阶段做第二次rename来提高性能。

在FileOutputCommitter V2文件提交机制中，如果部分task已执行成功，而此时job执行失败，就会出现一部分数据对外可见，也就是出现了脏数据，需要数据消费者根据是否新生成了_SUCCESS标识文件来判断数据的完整性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/91d3208e82f2428787be6b622c0c2fac.png)

常见的解决方法：1. 通过repartition/coalesce的方式，利用spark提供的参数实现小文件的合并

### 收益
- 减少文件数量，便于数据生命周期管理：对HDFS可以极大减小NameNode元数据管理的压力。对于AWS S3这类对象存储服务，其没有文件目录树这个概念，所有数据都在同一个层次，仅仅通过数据的唯一地址标识来识别并查找数据。基于这个特性，对于遍历目录下所有数据文件，AWS S3相比于HDFS性能要差上一个数量级。
- 提高用户作业的数据读取效率；
- 降低Spark driver端出现OOM的概率：对于Spark作业，数据源分布的小文件数量越多，则Spark作业在从数据源读取数据时，产生的task数量也会更多，而数据分片信息以及对应产生的task元信息都保存在driver端的内存中，会给driver带来很大的压力，甚至引发OOM异常导致作业执行失败。
- 加快大型Spark作业的执行速度。
# 英文
我在的团队是hulu广告智能团队下的data team，我们主要做的内容是负责hulu和disney的广告系统的数据仓库的搭建。21年底我们团队开始做一个核心项目就是给disney+平台上一套广告系统，之前disney是没有广告系统的。因此我们搭建一个数仓平台给广告业务提供支持。

整个广告的data处理过程可以分为 实时流式处理 和 离线批处理。实时流式处理的团队利用kafaka或者aws上knesis接受广告埋点数据，dump数据并且根据业务需求做一些实时处理。 我所在的 离线批处理团队 拿到实时团队 dump的广告埋点数据，对这些数据进行批量处理。 

我们的输入数据会包括从client端拿到的用户行为数据，比如用户观看行为，投放的广告有25%，50%还是75%被观看了，以及从sever端拿到的广告被填充的具体信息，广告的meta id等等。以及很多在聚合阶段需要使用到的meta信息。 

我们输出的数据是大宽表的形式被多个下游团队使用，比如BI 商业分析团队，广告归因团队，和算法团队。

具体的工作内容在业务层面上： 我们会对原始收集到的json数据进行 解析，归一化，校验。然后使用client端和sever端的事实数据 加上 很多的广告meta信息，用户meta信息进行聚合。我们最后得到的数据表主要是3个大的主题上，一个revenue主题的数据，我们会在单个广告的维度上进行聚合得到 ads delivery数据，也就是看看投放的广告被用户观看的记录。这些数据在经过进一步的 反欺诈反作弊的MRC以后，会被过滤掉一些异常流量，比如一个广告是否在在短时间被多次观看，是不是有一些非法的ip观看了广告之类的。  还有就是supply主题的数据，我们会统计在一个pod上被选取的广告的情况，这些信息会帮助广告选取和投放团队进行优化。 最后就是 auction biding竞价的层面，也是给到算法或者BI团队看看是否最大化了投放收益。


在技术层面上：对于数仓ELT的开发方面，我们组主要做的是离线处理业务，所以大数据处理框架选择是spark框架，底层的存储部署之类的技术是在AWS上搭建，比如数据存储实际上使用的是aws的s3存储，任务调度使用开源框架airflow去调度，数据监控用的grafana等。这部分我的工作就主要是实现之前的数据清洗，数据聚合，和异常流量过滤的代码。以及使用airflow框架搭建pipeline去调度任务，配合qa测试以及对spark job的性能进行调优。包括对pipeline配置一些警报之类的。以及需要做一些spark job的性能调优之类的工作。

I'm from the Data core Team under Hulu's Advertising Intelligence department. Our main responsibility is to build the data warehouse for Hulu and Disney's advertising systems. At the end of 2021, our team started to work on a core project, which was to provide an advertising system for the Disney+ platform. Disney did not have an advertising system before. Therefore, we built a data warehouse platform to provide support for the advertising business.

The data processing process of the entire advertisement can be divided into real-time streaming processing and offline batch processing. The real-time stream processing team uses kafaka or knesis on aws to receive advertising data, dump the data and do some real-time processing according to business requirement. The offline batch processing team I belong to, used the beacon data dumped by the real-time team, and performs batch processing on these data.

Our input data is divided into two categories. One is the user behavior data obtained from the client side, such as user viewing behavior, the user watched 25%, 50% or 75% of the advertisement. The other type is the advertisement information, which we can obtain from the server side, such as the classification of the advertisement, some ad pod information and so on.

The product we deliver is some wide tables and is used by multiple downstream teams, such as BI business analysis team, advertising attribution team, and algorithm team.


About the specific work content, in the business level: we will do some normalization, verification, unification, etc.  to the confrom raw data. Then we will use the fact data which we have confromed from client side and server side raw beacon and some advertisement meta information or user meta information to aggregate. 

The data table we finally deliver focus on three major topic. 

The first is revenue  data, we will aggregate on the dimension of a single ad to get the ads delivery data. Like get the statistics of the delivered ads which has been watched by users. These data need to be further processed by anti-cheating processed we called MRC, some abnormal traffic will be filtered out, such as whether an advertisement has been viewed multiple times in a short period of time, whether there are some illegal IPs viewing the advertisement and so on.

There is also the data of the supply topic. We will count the advertisements number which be selected in a pod. This information will help the advertisement selection and delivery team to optimize. 

Finally, there is the level of auction bidding, which is also given to the algorithm or BI team to see if the investment revenue is maximized.

On the technical level: our group mainly does offline data processing, so the big data processing framework is the spark framework. Storage and computing resources are built on AWS. For example, data storage actually uses AWS S3 storage, task scheduling uses the open source framework Airflow to schedule, and data monitoring uses Grafana, etc.



This part of my work is mainly to implement the previous data cleaning, data aggregation, and abnormal traffic filtering code. And use the airflow framework to build a pipeline to schedule tasks, cooperate with qa testing and optimize the performance of spark jobs. Including configuring some alarms on the pipeline and the like. And need to do some spark job performance tuning and other work.


# PII
This project is about the PII information usage permission control.

The US PM puts forward some requirements on the storage, management and use of users' personal information. I  need to communicate with the PM to clarify the requirements, and design the PII  permission control solution and implement it.

The main requirement is that we cannot store some PII data just in plain text, and we need to deal with RTD, and users have the right to delete the pii information. As well as PII data for users under the age of 13 may not be stored. Other users' data has a maximum storage period and needs to be deleted periodically.

This part of the work also needs to be combined with the requirement of some advertising businesses, because we will actually use some users' personal data to analyze user behavior. For example, analyzing user preferences and user profile. User information includes device-ad-id, ip address, and accout-id. For us, some data can be directly hashed, and we can also use it, such as ip and device id, because we mainly use it to compare with the ip and device on the blacklist and then perform traffic filtering. However, ip information is needed for some downstream teams. For example, the attribute team needs ip plaintext when doing advertising attribution.

plan:
1. Hash some content, such as device, ip, etc.
2. There is a compliance manager that can generate a key according to the user's accout_id. When we process data, we request the encrypted key for all users, and then, ip and other information that may need to be decrypted
and other information is encrypted.
1. For possible RTD cases, we do not delete in our data, but delete the data of the compliance manager. In this way, we can ensure that the data here does not appear inconsistency when calculating information such as bills.
2. For U13 content, we created a pipeline to delete these fields and moved them to the externally visible db.
3. Regular physical deletion of data exceeding 180 days






I'm from the Data core Team under Hulu's Advertising Intelligence department. Our main responsibility is to build the data warehouse for Hulu and Disney's advertising systems. Towards the end of 2021, our team started a major project to develop an advertising system for the Disney+ platform, which previously did not have an advertising system.

Specifically, our group is responsible for offline processing of the collected ad tracking data. This includes validating and normalizing the received factual data, as well as aggregating it with the ad meta information. This allows us to derive richer dimensional information related to ad revenue, ad fill rate, and ad bidding based on users' viewing behavior and ad factual information.

I joined the team in early 2022, coinciding with the phase when the team was clarifying project requirements and designing workflows. This involved collaborating with stakeholders to determine model fields, table structures, and specifying the specific data processing operations internally, such as field normalization and content validation. During this phase, I contributed to the design of data processing solutions.

Regarding the development of the data warehouse's Extract, Load, Transform (ELT) process, our team primarily focuses on offline business processing. Therefore, we chose the Spark framework for big data processing, and we deployed underlying storage technologies on AWS. For example, we utilized AWS S3 for data storage, the open-source framework Airflow for task scheduling, and Grafana for data monitoring. In this part, my main tasks included implementing code for data cleansing, data aggregation, and filtering abnormal traffic. Additionally, I worked on building pipelines using the Airflow framework to schedule tasks, collaborating with QA testing, and optimizing Spark jobs' performance. This also involved configuring alerts for the pipeline.

For Spark performance tuning, our primary concerns were ensuring that jobs run smoothly, monitoring their execution time, and verifying that resource allocation is reasonable.    