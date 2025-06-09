# Spark学习笔记

## 尚硅谷2024-Spark

### （1）Spark概述

- Spark基于MR开发，前者Scala语言后者Java语言，Scala更适用于函数式的封装与调用，适合处理大量的数据，Java是面向对象的语言，强调方法，不适合进行大量的数据处理。
- Hadoop出现早，只考虑单一的计算操作，若需要重复操作无法迭代，会造成大量的磁盘读写操作（OI）；Spark优化了计算过程，中间计算结果存于内存（不进入磁盘），基于内存上的性能提升（也会受到内存的限制）。MR基于磁盘，尽管慢，但是“稳定”。
- Spark组件：Spark Core（核心通用）、Spar SQL（结构化查询）、Spark Streaming（实时计算）、Spark Mlib（机器学习）、Spark Graph X（图计算）
- Hadoop 2.x 出现YARN，之前MR计算和资源管理耦合在一起，无法摈弃MR；而在YARN解耦出来之后，就可以用其他他计算框架替换MapReduce，比如Spark、Flink
- 部署环境：YARN环境或者Spark环境（自身也有资源管理）；在实际工程领域采用“YARN+Spark”即“Spark ON YARN”

### （2）SparkRDD

- 凡是源代码解析的一律跳过，但凡是部署的一律跳过，转而认真去理解Spark基础知识、
- RDD基础知识与原理、转换算子和行动算子基本知识、groupBy算子与reduceByKey算子的区别与联系、
- 依赖关系与血缘关系、宽窄依赖的基础知识、Application&Job&Stage&Task的关系、RDD的持久化（Cache、Persist、RDD CheckPoint检查点）；
- 今天学习进度就先到这，下一步：99-广播变量。

### （3）SparkSQL

- 深入浅出的理解SQL到底层任务的转换执行逻辑`SQL->Hive->MR->jar->run->JVM->Process->main`，MR运行效率慢所以用Spark代替，MR开发效率慢所以用Hive代替；Spark+Hive->Shark，但是由于Spark与Hive版本迭代速度不一致，进而衍生出`Spark on Hive(Spark自身来解析SQL，Hive作为数据源)`（即SparkSQL）和`Hive on Spark(Hive来解析SQL)`两大方向。

- 讲解的太底层了，而且我个人没有Java基础，所以源码讲解部分基本上都跳过了（大部分都是源码讲解）

- SparkSQL案例分析SQL实现3讲的很详细了（对于GROUP BY的讲解），也和我之前学习的理解的用法一致。

### （4）SparkStreaming

- SparkStreaming是对于SparkCore（RDD）的封装，适用于特定场合，实际上就是将无界数据流切分为有界数据流。

- SparkStreaming**微批量**、**准实时**的数据处理框架（一小批数据一小批数据的处理、数据处理延迟以秒和分钟为单位）

- DStream 是 Spark Streaming 的基础抽象，代表持续性的数据流和经过各种 Spark 算子操作后的**结果数据流**。
  - 在内部实现上，每一批次的数据封装成一个 RDD，一系列连续的 RDD 组成了 DStream。对这些 RDD 的转换是由 Spark 引擎来计算。
  - 说明：DStream 中批次与批次之间计算相互独立。如果批次设置时间小于计算时间会出现计算任务叠加情况，需要多分配资源。通常情况，**批次设置时间要大于计算时间**。
  - Window Operations 可以设置窗口的大小和滑动窗口的间隔来动态的获取当前 Streaming 的允许状态。所有基于窗口的操作都需要两个参数，分别为**窗口时长**以及**滑动步长**。
    - ➢ 窗口时长：计算内容的时间范围
    - ➢ 滑动步长：隔多久触发一次计算
    - 注意：这两者都必须为采集批次大小的整数倍。如下图所示 WordCount 案例：窗口大小为批次的 2 倍，滑动步等于批次大小。

- 区分一下算子、方法、原语

- 

- 

- 

- 

- 

### （5）SparkCore（内核）
Spark的内存管理其实是基于Java的内存管理

- 堆外内存：我的理解就是JVM没有“完全权限”的内存（不在JVM堆内管理的内存区域）

- 堆内内存：我的理解就是JVM拥有“完全权限”的内存，比如下达GC命令（垃圾回收）会立刻执行
