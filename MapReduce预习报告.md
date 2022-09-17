MapReduce预习报告
1、MapReduce是什么？
MapReduce是一种用于在大型商用硬件集群中（成千上万的结点）对海量数据（多个兆字节数据集）实施可靠的、高容错的分布式计算的框架，也是一种经典的并行计算模型。MapReduce的基本原理是将一个复杂的问题（数据集）分成若干个简单的子问题（数据块）进行解决（Map函数）；然后对子问题的结果进行合并（Reduce函数），得到原有问题的解（结果）。同时MapReduce模型适合于大文件的处理，对很多小文件的处理效率不是很高，这一点和HDFS一样。

2、MapReduce编程模型简介
（1）MapReduce简单模型
对于某些任务来说，可能并不一定需要Reduce过程，如只需对文本的每一行数据作简单的格式转换即可，那么只需要由Mapper处理后就可以了。所以MapReduce也有简单的编程模型，该模型只有Mapper过程，由Mapper产生的数据直接写入HDFS。
（2）MapReduce复杂模型
对于大部分的任务来说，都是需要Reduce过程，并且由于任务繁重，会启动多个Reduce（默认为1，根据任务量可由用户自己设定合适的Reduce数量）来进行汇总。如果只用一个Reduce计算所有Mapper的结果，会导致单个Reduce负载过于繁重，成为性能的瓶颈，大大增加任务的运行周期。

3、MapReduce数据流
（1）分片、格式化数据源（InputFormat）
InputFormat主要有两个任务，一个是对源文件进行分片，并确定Mapper的数量；另一个是对各分片进行格式化，处理成<key,value>形式的数据流并传给Mapper。
(2)Map过程
Mapper接收<key,value>形式的数据，并处理成<key,value>形式的数据，具体的处理过程可由用户定义。
（3）Combiner过程
每一个map（）都可能会产生大量的本地输出，Combiner（）的作用就是对map（）端的输出先做一次合并，以减少在Map和Reduce结点之间的数据传输量，提高网络I/O性能，是MapReduce的一种优化手段之一。
（4）Shuffle过程
Shuffle过程是指从Mapper产生的直接输出结果，经过一系列的处理，成为最终的Reduce直接输入数据为止的整个过程，这一过程也是MapReduce的核心过程。
（5）Reduce过程
Reduce接收<key,{value list}>形式的数据流，形成<key,value>形式的数据输出，输出数据直接写入HDFS，具体的处理过程可由用户定义。

4、MapReduce任务运行流程
MapReduce的任务流程是从客户端提交任务开始，直到任务运行结束的一系列流程。MRv2是Hadoop2中的MapReduce任务运行流程。在MRv2中，MapReduce运行时环境由Yarn提供，所以需要MapReduce相关服务进行协同工作。
（1）MRv2基本组成
a.客户端（client）
b.MRAppMaster
c.Map Task和Reduce Task
（2）Yarn基本组成
a.Resource Manager（RM）
b.NodeManager
c.ApplicationMaster(AM)
d.container
（3）任务流程
a.client向ResourceManager提交任务。
b.ResourceManager分配该任务的第一个container，并通知相应的NodeManager启动MRAppMaster。
c.NodeManager接收命令后，开辟一个container资源空间，并在container中启动相应的MRAppMaster。
d.MRAppMaster启动之后，第一步会向ResourceManager注册，这样用户可以直接通过MRAppMaster监控任务的运行状态；之后则直接由MRAppMaster调度任务运行，重复（e~h）,直到任务结束。
e.MRAppMaster以轮询的方式向ResourceManager申请任务运行所需的资源。
f.一旦ResourceManager配给了资源，MRAppMaster便会与相应的NodeManager通信，让它划分Container并启动相应的任务（Map Task或Reduce Task）。
g.NodeManager准备好运行环境，启动任务。
h.各任务运行，并定时通过RPC协议向MRAppMaster汇报自己的运行状态和进度。MRAppMaster也会实时地监控任务的运行，当发现某个Task假死或失败时，便杀死它重新启动任务。
i.任务完成，MRAppMaster向ResourceManager通信，注销并关闭自己。
