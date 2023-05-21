# Tech sharing
## airflow
Workflow management platform for data engineering pipeline
Programmatically author, schedule and monitor existing tasks

> concepts

- DAG: Scheduler looks at DAG’s schedules and kick off a DAG run
- Operator: Operator is a template for the task, like the operator will submit a spark job. 
- Task: This task is going to be this operator with parameters. You are taking the operator and instantiate into a task

Dag runs turns into a DagRun, all those tasks which are instances of operators then become task instances

> database info

- Dag info: describes an instance of dag, execution_date, start_date, end_date
- Task info: task_id, dag_id, job_id, execution_date, start_date, end_date

> 生命周期

> component

- Scheduler: work out what task instance need to run.
- Executor: run task and record result.


# Presto
presto可以跨数据源访问，包括完包括一个 Coordinator 和多 个 Worker。由客户端提交查询，从 Presto 命令行 CLI 提交到 Coordinator。Coordinator 进行 解析，分析并执行查询计划，然后分发处理队列到 Worker。通信是通过 REST API。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b33f7181f4c4dffba7b332d37a6a67e.png)

Presto 处理 table 时，是通过表的完全限定（fully-qualified）名来找到 catelog。例如， 一个表的权限定名是 hive.test_data.test，则 test 是表名，test_data 是 schema，hive 是 catelog。

Presto 与 Hive 对比，都能够处理 PB 级别的海量数据分析，但 Presto 是基于内存运算，减少没必要的硬盘 IO，所以更快。

虽然能够处理 PB 级别的海量数据分析，但不是代表 Presto 把 PB 级别都放在内存中计算的。而是根据场景，如 count，avg 等聚合运算，是边读数据边计算，再清内存，再读数据再计算，这种耗的内存并不高。但是连表查，就可能产生大量的临时数据，因此速度会变慢，反而 Hive此时会更擅长。

  Presto 对 ORC文件 读取进行了特定优化，因此，在 Hive 中创建 Presto 使用的表时，建议采用 ORC 格式存储。相对于 Parquet 格式，Presto 对 ORC 格式支持得更好。

## Presto 的存储单元
Presto中处理的最小数据单元是一个Page对象，Page对象的数据结构如下图所示。一个Page对象包含多个Block对象，每个Block对象是一个字节数组，存储一个字段的若干行。多个Block横切的一行是真实的一行数据。一个Page最大1MB，最多16*1024行数据。

- Page：多行数据的集合，包含多个列的数据，内部仅提供逻辑行，实际以列式存储。
- Block：一列数据，根据不同类型的数据，通常采取不同的编码方式，了解这些编码方式，有助于自己的存储系统对接 presto

**对比数据仓库，dwd层建议不要使用ORC，而dm层则建议使用。**

# 列式存储
由于OLAP查询的特点，列式存储可以提升其查询性能，但是它是如何做到的呢？这就要从列式存储的原理说起，从图1中可以看到，相对于关系数据库中通常使用的行式存储，在使用列式存储时每一列的所有元素都是顺序存储的。由此特点可以给查询带来如下的优化：

- 查询的时候不需要扫描全部的数据，而只需要读取每次查询涉及的列，这样可以将I/O消耗降低N倍，另外可以保存每一列的统计信息(min、max、sum等)，实现部分的谓词下推。
- 由于每一列的成员都是同构的，可以针对不同的数据类型使用更高效的数据压缩算法，进一步减小I/O。可以使用更加适合CPU pipeline的编码方式，减小CPU的缓存失效。
 
## ORC
![在这里插入图片描述](https://img-blog.csdnimg.cn/01649e4e1e234b6eb6ad45f8d2622e2c.png)

- stripe：ORC文件存储数据的地方，每个stripe一般为HDFS的块大小。（包含以下3部分）
  - index data:保存了所在条带的一些统计信息,以及数据在 stripe中的位置索引信息。
  - rows data:数据存储的地方,由多个行组构成，每10000行构成一个行组，数据以流( stream)的形式进行存储。
  - stripe footer:保存数据所在的文件目录
- 文件脚注( file footer)：包含了文件中sipe的列表,每个 stripe的行数,以及每个列的数据类型。它还包含每个列的最小值、最大值、行计数、求和等聚合信息。
- postscript：含有压缩参数和压缩大小相关的信息
## Parquet
>parquet存储格式
列式存储。支持Parquet支持嵌套的数据模型。

类似于Protocol Buffers，每一个数据模型的schema包含多个字段，每一个字段有三个属性：重复次数、数据类型和字段名，重复次数可以是以下三种：required(只出现1次)，repeated(出现0次或多次)，optional(出现0次或1次)。每一个字段的数据类型可以分成两种：group(复杂类型)和primitive(基本类型)。

```txt
message Document {
  required int64 DocId;
  optional group Links {
    repeated int64 Backward;
    repeated int64 Forward; 
  }
  repeated group Name {
    repeated group Language {
      required string Code;
      optional string Country; 
     }
    optional string Url; 
  }
}
```


>parquet文件结构
Parquet文件是以二进制方式存储的，是不可以直接读取和修改的，Parquet文件是自解析的，文件中包括该**文件的数据和元数据**。在HDFS文件系统和Parquet文件中存在如下几个概念：

- HDFS块(Block)：它是HDFS上的最小的副本单位，HDFS会把一个Block存储在本地的一个文件并且维护分散在不同的机器上的多个副本，通常情况下一个Block的大小为256M、512M等。
- HDFS文件(File)：一个HDFS的文件，包括数据和元数据，数据分散存储在多个Block中。
  
- 行组(Row Group)：按照行将数据物理上划分为多个单元，每一个行组包含一定的行数，在一个HDFS文件中至少存储一个行组，Parquet读写的时候会将整个行组缓存在内存中，所以如果每一个行组的大小是由内存大的小决定的。
- 列块(Column Chunk)：在一个行组中每一列保存在一个列块中，行组中的所有列连续的存储在这个行组文件中。不同的列块可能使用不同的算法进行压缩。
- 页(Page)：每一个列块划分为多个页，一个页是最小的编码的单位，在同一个列块的不同页可能使用不同的编码方式。

## 选型
- parquet对于复杂嵌套类型的支持比较好；
- spark更支持parquet，做了更多的优化。在减小文件大小方面有一些优化。
- ORC支持ACID和update操作。
- ORC的压缩性能更好

# 数仓

## 离线数仓架构-lambda/kapaa
### lambda
离线和实时处理技术走两条线，离线的专门做离线数据处理（例如使用hive，impala，presto，sparkSQL等各种OLAP的技术框架），实时的就专门使用实时处理技术（例如storm，sparkStreaming，flink流处理程序等）。

一条线是进入流式计算平台（例如 Storm、Flink或者Spark Streaming），去计算实时的一些指标；另一条线进入批量数据处理离线计算平台（例如Mapreduce、Hive，Spark SQL），去计算T+1的相关业务指标，这些指标需要隔日才能看见。

>存在的问题

- 实时与批量计算结果不一致引起的数据口径问题：因为批量和实时计算走的是两个计算框架和计算程序，算出的结果往往不同，经常看到一个数字当天看是一个数据，第二天看昨天的数据反而发生了变化。
- 批处理和流处理代码不统一，需要额外的开发成本。
服务器存储大：数据仓库的典型设计，会产生大量的中间结果表，造成数据急速膨胀，加大服务器存储压力。

![在这里插入图片描述](https://img-blog.csdnimg.cn/29e4175b13de4cb19d2d9c14910c0016.png)


随着业务的发展，随着业务的发展，人们对数据实时性提出了更高的要求。此时，出现了Lambda架构，其将对实时性要求高的部分拆分出来，增加条实时计算链路。从源头开始做流式改造，将数据发送到消息队列中，实时计算引擎消费队列数据，完成实时数据的增量计算。与此同时，批量处理部分依然存在，实时与批量并行运行。最终由统一的数据服务层合并结果给于前端。一般是以批量处理结果为准，实时结果主要为快速响应。

### kappa架构
在 Kappa 架构中，用流式处理解决问题，对于批处理的数据修正问题可以通过数据重放解决。需求修改或历史数据重新处理都通过上游重放完成。

- 用Kafka或者类似MQ队列系统收集各种各样的数据，需要几天的数据量就保存几天。
- 当需要全量重新计算时，重新起一个流计算实例，从头开始进行处理，并输出到一个新的结果存储中。
- 当新的实例做完后，停止老的流计算实例，并把老的一些结果删除。

> 缺点

- 吞吐能力不足，通过加资源只能解决一部分的问题。
- 中间件的缓存问题，需要有比较好的缓存能力。
- 在实时数据处理时，遇到大量不同的实时流进行关联时，非常依赖实时计算系统的能力，很可能因为数据流先后顺序问题，导致数据丢失。 比如广告领域的 request和event match问题。


## 数仓分层
- ODS，操作数据层，保存原始数据；
- DW 数据仓库层
  - DWD，数据仓库明细层，根据主题定义好事实与维度表，保存最细粒度的事实数据；是对ODS层的数据进行清洗后提取的出来的
  - DWS 而DWS层是经过了一些轻度汇总后的数据。也可以叫DM，数据集市/轻度汇总层，
- ADS层则是产出应用最终所需的数据。

lambda架构下的数仓结构，包括了一个实时明细层，存放了一些实时数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0276649b8b844e589decabe059ddb583.png)


# ODS structure
>ods logic view
![在这里插入图片描述](https://img-blog.csdnimg.cn/81163db927474aae9c4a08752032ff43.png)

>ods implement view
![在这里插入图片描述](https://img-blog.csdnimg.cn/176e6137ee7446c19e188dedcfb36ddc.png)

>ods product overview
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c213fa27d184371a5085ccdc570befc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb3e9c07c71242eaadca879dd3ffe9b2.png)

>data workflow
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e6d5bad0f034bfa8ba2ad3896c25f11.png)

1.数仓
Situation: 数仓规模PB，技术栈（aws s3+glue），event
Target: source of true data
理解广告业务，梳理数据模型，沟通， 
广告revenue, supply, autction 3 subjects 维度建模，搭建ods层，dwd明细层，ads
2.Spark ELT
Sitation: 大量离线数据需要处理，source of true.
Target: 高效，高质量的输出etl后的数据
Spark 调优（传统），解决云上spark低效问题 (s3)
数据质量
pipeline
3.migration
Yarn spark -> eks spark, hads -> s3
老的piepline优化
1.简洁，2.解偶
3.稳定，4.幂等 更好维护
4.数据安全
Situation： 谁能访问，存什么数据(PII)不能存
Action result: 设计了pii的方案，保护用户隐私 (包装)