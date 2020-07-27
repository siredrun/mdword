Hadoop是一个由Apache基金会所开发的分布式系统基础架构。

用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。

Hadoop实现了一个分布式文件系统（Hadoop Distributed File System），简称HDFS。HDFS有高容错性的特点，并且设计用来部署在低廉的（low-cost）硬件上；而且它提供高吞吐量（high throughput）来访问应用程序的数据，适合那些有着超大数据集（large data set）的应用程序。HDFS放宽了（relax）POSIX的要求，可以以流的形式访问（streaming access）文件系统中的数据。

Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，而MapReduce则为海量的数据提供了计算。 

# 概述

## 三大发行版本

Hadoop三大发行版本：Apache、Cloudera、Hortonworks。

Apache版本最原始（最基础）的版本，对于入门学习最好。

Cloudera在大型互联网企业中用的较多。

Hortonworks文档较好。

Apache Hadoop

官网地址：http://hadoop.apache.org/releases.html

下载地址：https://archive.apache.org/dist/hadoop/common/

Cloudera Hadoop 

官网地址：https://www.cloudera.com/downloads/cdh/5-10-0.html

下载地址：http://archive-primary.cloudera.com/cdh5/cdh/5/

（1）2008年成立的Cloudera是最早将Hadoop商用的公司，为合作伙伴提供Hadoop的商用解决方案，主要是包括支持、咨询服务、培训。

（2）2009年Hadoop的创始人Doug Cutting也加盟Cloudera公司。Cloudera产品主要为CDH，Cloudera Manager，Cloudera Support

（3）CDH是Cloudera的Hadoop发行版，完全开源，比Apache Hadoop在兼容性，安全性，稳定性上有所增强。

（4）Cloudera Manager是集群的软件分发及管理监控平台，可以在几个小时内部署好一个Hadoop集群，并对集群的节点及服务进行实时监控。Cloudera Support即是对Hadoop的技术支持。

（5）Cloudera的标价为每年每个节点4000美元。Cloudera开发并贡献了可实时处理大数据的Impala项目。

\3. Hortonworks Hadoop

官网地址：https://hortonworks.com/products/data-center/hdp/

下载地址：https://hortonworks.com/downloads/#data-platform

（1）2011年成立的Hortonworks是雅虎与硅谷风投公司Benchmark Capital合资组建。

（2）公司成立之初就吸纳了大约25名至30名专门研究Hadoop的雅虎工程师，上述工程师均在2005年开始协助雅虎开发Hadoop，贡献了Hadoop80%的代码。

（3）雅虎工程副总裁、雅虎Hadoop开发团队负责人Eric Baldeschwieler出任Hortonworks的首席执行官。

（4）Hortonworks的主打产品是Hortonworks Data Platform（HDP），也同样是100%开源的产品，HDP除常见的项目外还包括了Ambari，一款开源的安装和管理系统。

（5）HCatalog，一个元数据管理系统，HCatalog现已集成到Facebook开源的Hive中。Hortonworks的Stinger开创性的极大的优化了Hive项目。Hortonworks为入门提供了一个非常好的，易于使用的沙盒。

（6）Hortonworks开发了很多增强特性并提交至核心主干，这使得Apache Hadoop能够在包括Window Server和Windows Azure在内的Microsoft Windows平台上本地运行。定价以集群为基础，每10个节点每年为12500美元。

## 优势（4高）

1.高可靠性：底层维护多个数据副本，即使Hadoop某个计算元素或存储出现故障，也不会导致数据丢失。

2.高扩展性：在集群间分配数据任务数据，可方便地扩展到数以千计的节点中。

3.高效性：在MapReduce思想下，Hadoop是并行工作的，以加快任务处理熟读

4.高容错性。能够自动将失败的任务重新分配。

## 组成（重点）

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image002.jpg) 

HDFS（Hadoop Distributed File System）Hadoop分布式文件系统架构概述。

1）NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。类比武功目录。

2）DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。类比武功秘籍。

3）Secondary NameNode(2nn)：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。类比监控中心。

YARN架构概述。

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image004.jpg)

图2-24 YARN架构概述

MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce，如图2-25所示

1）Map阶段并行处理输入数据

2）Reduce阶段对Map结果进行汇总

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image006.jpg) 

## 大数据技术生态体系

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image008.jpg) 

图中涉及的技术名词如下：

1）Sqoop：Sqoop是一款开源的工具，主要用于在Hadoop、Hive与传统的数据库(MySql)间进行数据的传递，可以将一个关系型数据库（例如 ：MySQL，Oracle 等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

2）Flume：Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

3）Kafka：Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：

（1）通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。

（2）高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息。

（3）支持通过Kafka服务器和消费机集群来分区消息。

（4）支持Hadoop并行数据加载。

4）Storm：Storm用于“连续计算”，对数据流做连续查询，在计算时就将结果以流的形式输出给用户。

5）Spark：Spark是当前最流行的开源大数据内存计算框架。可以基于Hadoop上存储的大数据进行计算。

6）Oozie：Oozie是一个管理Hdoop作业（job）的工作流程调度管理系统。

7）Hbase：HBase是一个分布式的、面向列的开源数据库。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。

8）Hive：Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。

10）R语言：R是用于统计分析、绘图的语言和操作环境。R是属于GNU系统的一个自由、免费、源代码开放的软件，它是一个用于统计计算和统计制图的优秀工具。

11）Mahout：Apache Mahout是个可扩展的机器学习和数据挖掘库。

12）ZooKeeper：Zookeeper是Google的Chubby一个开源的实现。它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、 分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

# Hadoop运行环境搭建

虚拟机环境准备

创建并克隆一个centos7 64位虚拟机，并使用Xshell通过root用户连接，记得关闭防火墙：

```
service firewalld stop 
```

最后再配置禁用防火墙，然后重启虚拟机防火墙关闭才不会失效

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```



## 安装JDK

Hadoop Java版本。Apache Hadoop的2.7版和更高版本需要Java7。它是在OpenJDK和Oracle（HotSpot）的JDK / JRE上构建和测试的。早期版本（2.6和更早版本）支持Java 6。Apache Hadoop 3.x现在仅支持Java 8
从2.7.x到2.x的Apache Hadoop支持Java 7和8，Java 11支持正在进行中（截止2020-03-17）。

卸载现有JDK

（1）查询是否安装Java软件：

```
rpm -qa | grep java
```

（2）卸载原有的JDK：

```
sudo rpm -e 软件包名
```

（3）查看JDK安装路径：

```
which java
```

下载jdk-8u144-linux-x64.tar.gz并用Xftp传入到Linux系统下的/opt/software下

\1. 解压JDK到/opt目录下：

```
tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
```

2  配置JDK环境变量

（1）先获取JDK路径

```
 pwd   /opt/jdk1.8.0_144
```

（2）打开/etc/profile文

```
sudo vi /etc/profile
```

在profile文件末尾添加JDK路径

```
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
```

（3）保存后退出:wq

（4）让修改后的文件生效：source /etc/profile

\6. 测试JDK是否安装成功： java -version

注意：重启（如果java -version可以用就不用重启）：sudo reboot

## 安装Hadoop

Hadoop下载地址：https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz

1.用XFTP工具将hadoop-3.2.1.tar.gz导入到/opt文件夹下

2.进入到Hadoop安装包路径下：

```
cd /opt/
```

3.解压安装文件到/opt下面：

```
tar -zxvf hadoop-2.7.2.tar.gz
```

4.查看是否解压成功：

```
ls /opt/
```

更名：

```
mv hadoop-3.2.1 hadoop
```

5.将Hadoop添加到环境变量

（1）获取Hadoop安装路径：pwd   /opt/hadoop

（2）打开/etc/profile文件：

```
sudo vim /etc/profile
```

在profile文件末尾添加Hadoop路径：（shitf+g到达最底部）

```
export JAVA_HOME=/opt/java
export HADOOP_HOME=/opt/hadoop
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${JAVA_HOME}/bin:$PATH
```

（3）保存后退出:wq

（4）让修改后的文件生效：

```
source /etc/profile
```

6.测试是否安装成功：

```
hadoop version
```

效果：

```
Hadoop 3.2.1
Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r b3cbbb467e22ea829b3808f4b7b01d07e0bf3842
Compiled by rohithsharmaks on 2019-09-10T15:56Z
Compiled with protoc 2.5.0
From source with checksum 776eaf9eee9c0ffc370bcbc1888737
This command was run using /opt/hadoop/share/hadoop/common/hadoop-common-3.2.1.jar
```

7.重启(如果Hadoop命令不能用再重启)

```
 sudo reboot
```

## 目录结构

1、查看Hadoop目录结构：

```
ll /opt/hadoop/
```

内容：

```

总用量 180
drwxr-xr-x. 2 1001 1001    203 9月  11 2019 bin
drwxr-xr-x. 3 1001 1001     20 9月  10 2019 etc
drwxr-xr-x. 2 1001 1001    106 9月  11 2019 include
drwxr-xr-x. 3 1001 1001     20 9月  11 2019 lib
drwxr-xr-x. 4 1001 1001    288 9月  11 2019 libexec
-rw-rw-r--. 1 1001 1001 150569 9月  10 2019 LICENSE.txt
-rw-rw-r--. 1 1001 1001  22125 9月  10 2019 NOTICE.txt
-rw-rw-r--. 1 1001 1001   1361 9月  10 2019 README.txt
drwxr-xr-x. 3 1001 1001   4096 9月  10 2019 sbin
drwxr-xr-x. 4 1001 1001     31 9月  11 2019 share
```

2、重要目录

（1）bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本

（2）etc目录：存放Hadoop配置文件

（3）lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）

（4）sbin目录：存放启动或停止Hadoop相关服务的脚本

（5）share目录：存放Hadoop的依赖jar包、文档、和官方案例

# Hadoop运行模式

Hadoop运行模式包括：本地模式、伪分布式模式以及完全分布式模式。

Hadoop官方网站：http://hadoop.apache.org/

## 本地运行模式

### 官方Grep案例

1.创建在hadoop文件下面创建一个input文件夹：

```
sudo mkdir /opt/hadoop/input
```

2.将Hadoop的xml配置文件复制到input：

```
cp /opt/hadoop/etc/hadoop/*.xml /opt/hadoop/input/
```

3.执行share目录下的MapReduce程序：

```
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep /opt/hadoop/input/ /opt/hadoop/output 'dfs[a-z.]+'
```

注意：input输入文件，output输出文件，dfs[a-z.]+正则表达式，执行这个命令会自动创建output文件夹，如果此项目下原来就有output文件夹可能会报FileAlreadyExistsException，如报此异常，使用rm -rf output删除output文件即可。

打印内容：

```
020-03-17 03:21:40,652 INFO impl.MetricsConfig: Loaded properties from hadoop-metrics2.properties
2020-03-17 03:21:46,959 INFO impl.MetricsSystemImpl: Scheduled Metric snapshot period at 10 second(s).
2020-03-17 03:21:46,959 INFO impl.MetricsSystemImpl: JobTracker metrics system started
2020-03-17 03:21:53,657 INFO input.FileInputFormat: Total input files to process : 9
2020-03-17 03:21:53,689 INFO mapreduce.JobSubmitter: number of splits:9
2020-03-17 03:21:53,877 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local2099571503_0001
2020-03-17 03:21:53,877 INFO mapreduce.JobSubmitter: Executing with tokens: []
2020-03-17 03:21:54,032 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
2020-03-17 03:21:54,033 INFO mapreduce.Job: Running job: job_local2099571503_0001
2020-03-17 03:21:54,037 INFO mapred.LocalJobRunner: OutputCommitter set in config null
2020-03-17 03:21:54,049 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 2
2020-03-17 03:21:54,050 INFO output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2020-03-17 03:21:54,052 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
2020-03-17 03:21:54,105 INFO mapred.LocalJobRunner: Waiting for map tasks
...
```

4.查看输出结果：

```
cat /opt/hadoop/output/* 或 cat /opt/hadoop/output/part-r-00000
```

内容：

```
1	dfsadmin
```

dfsadmin  #正则表达式的意思是找出所有带dfs的字符

### 官方WordCount案例（常用）

1.在hadoop文件下面创建一个wcinput文件夹：

```
sudo mkdir /opt/hadoop/wcinput
```

2.在wcinput文件下创建一个wc.input文件

```
vim /opt/hadoop/wc.input
```

并输入如下内容

```
hadoop yarn
hadoop mapreduce
root
root
```

保存退出：wq

3.执行程序：

```
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /opt/hadoop/wcinput/ /opt/hadoop/wcoutput
```

6.查看结果：

```
cat /opt/hadoop/wcoutput/part-r-00000  #打印出所有单词出现的次数
```

打印：

```
[root@root hadoop]# 
hadoop	2
mapreduce	1
root	2
yarn	1
```



## 伪分布式运行模式

#### 启动HDFS并运行MapReduce程序

分析

（1）配置集群

（2）启动、测试集群增、删、查

（3）执行WordCount案例

执行步骤

##### 配置集群

（a）配置hadoop-env.sh：

Linux系统中获取JDK的安装路径：

```
echo $JAVA_HOME
```

编辑hadoop-env.sh：

```
vim /opt/hadoop/etc/hadoop/hadoop-env.sh
```

修改JAVA_HOME 路径：

```
export JAVA_HOME=/opt/java
```

（b）配置：

```
vim /opt/hadoop/etc/hadoop/core-site.xml
```

把下面内容复制到xml文件里的<configuration>标签里

```
<!-- 指定HDFS中NameNode的地址 -->
<property>
<name>fs.defaultFS</name>
    <value>hdfs://192.168.126.128:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/hadoop/data/tmp</value>
</property>
```

注意：192.168.126.128是自己的虚拟机地址

（c）配置：hdfs-site.xml：

```
vim /opt/hadoop/etc/hadoop/hdfs-site.xml
```

这个值默认为1，也是复制到xml文件里的<configuration>标签里

```
<!-- 指定HDFS副本的数量 -->
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>
```

##### 启动集群

（a）格式化NameNode（第一次启动时格式化，以后就不要总格式化）

如果需要格式化，1.通过jps把查出的namenode和datanode进程杀掉。2.删除data和logs文件夹

命令：

```
hdfs namenode -format
```

（b）启动NameNode

命令：

```
./opt/hadoop/sbin/hadoop-daemon.sh start namenode
```

可换成hdfs --daemon start namenode

（c）启动DataNode

命令：

```
./opt/hadoop/sbin/hadoop-daemon.sh start datanode
```

可换成

```
hdfs --daemon start datanode
```

（3）查看集群

（a）查看是否启动成功：jps

```
31360 NameNode
31568 Jps
31490 DataNode
```

注意：jps是JDK中的命令，不是Linux命令。不安装JDK不能使用jps

（b）web端查看HDFS文件系统[http://192.168.247.128:50070](http://192.168.247.128:50070/)

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image004-1584383932165.jpg)

注意：如果不能访问，看如下帖子处理

http://www.cnblogs.com/zlslch/p/6604189.html

（c）查看产生的Log日志

说明：在企业中遇到Bug时，经常根据日志提示信息去分析问题、解决Bug。

当前目录：cd /opt/module/hadoop-2.7.2/logs

[atguigu@hadoop101 logs]$ ls

  -rw-r--r--. 1 root root 98552 11月 6 11:12 hadoop-root-datanode-sire.log  -rw-r--r--. 1 root root  714 11月 6 11:05 hadoop-root-datanode-sire.out  -rw-r--r--. 1 root root  714 11月 6 11:01 hadoop-root-datanode-sire.out.1  -rw-r--r--. 1 root root  714 11月 6 10:34 hadoop-root-datanode-sire.out.2  -rw-r--r--. 1 root root  714 11月 6 10:17 hadoop-root-datanode-sire.out.3  -rw-r--r--. 1 root root 116878 11月 6 11:05  hadoop-root-namenode-sire.log  -rw-r--r--. 1 root root  5022 11月 6 11:14 hadoop-root-namenode-sire.out  -rw-r--r--. 1 root root  714 11月 6 11:00 hadoop-root-namenode-sire.out.1  -rw-r--r--. 1 root root  714 11月 6 10:33 hadoop-root-namenode-sire.out.2  -rw-r--r--. 1 root root  714 11月 6 10:16 hadoop-root-namenode-sire.out.3  -rw-r--r--. 1 root root   0 11月 6 10:16 SecurityAuth-root.audit  

查看日志：cat hadoop-root-datanode-sire.log #root表示当前Linux用户，sire表示Linux系统设置名。

（d）思考：为什么不能一直格式化NameNode，格式化NameNode，要注意什么？

注意：格式化NameNode，会产生新的集群id,导致NameNode和DataNode的集群id不一致，集群找不到已往数据。所以，格式NameNode时，一定要先删除data数据和log日志，然后再格式化NameNode。

注意：data和name的clusterID要一致

命令：cat /opt/module/hadoop-2.7.2/data/tmp/dfs/name/current/VERSION

  #Wed Nov 06 14:20:57 CST 2019  namespaceID=1863748626  **clusterID=CID-1f4d39c7-08dc-4459-abb7-d772cc43b0da**  cTime=0  storageType=NAME_NODE  blockpoolID=BP-2144810134-192.168.247.128-1573021257132  layoutVersion=-63  

命令：cat /opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/VERSION

  #Wed Nov 06 14:23:43 CST 2019  storageID=DS-51e5676b-6119-43ce-98ee-6ce0f2f30438  **clusterID=CID-1f4d39c7-08dc-4459-abb7-d772cc43b0da**  cTime=0  datanodeUuid=f91c2bc7-68ff-4360-b4d9-b1427a384d66  storageType=DATA_NODE  layoutVersion=-56  

##### 操作集群

（a）在HDFS文件系统上创建一个input文件夹：bin/hdfs dfs -mkdir -p /usr/root/input

（b）将测试文件内容上传到文件系统上：bin/hdfs dfs -put wcinput/wc.input /usr/root/input

（c）查看上传的文件是否正确：bin/hdfs dfs -lsr /usr/root/input 或 bin/hdfs dfs -cat /usr/root/input/wc.input

  doop yarn  hadoop mapreduce  root  root  

（d）运行MapReduce程序：bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /usr/root/input /usr/root/output

（e）查看输出结果：bin/hdfs dfs -cat /usr/root/output/p*

  doop 1  hadoop   1  mapreduce    1  root 2  yarn 1  

浏览器查看http://192.168.247.128:50070/explorer.html#/usr/root/output

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image006-1584383932165.jpg) 

（f）将测试文件内容下载到本地：bin/hdfs dfs -get /usr/root/output/part-r-00000 wcoutput

[root@sire hadoop-2.7.2]# bin/hdfs dfs -rm -rf /usr/root/output 

（g）删除输出结果：bin/hdfs dfs -rm -r /usr/root/output

#### 启动YARN并运行MapReduce程序

分析

（1）配置集群在YARN上运行MR

（2）启动、测试集群增、删、查

（3）在YARN上执行WordCount案例

执行步骤    

##### 配置集群

（a）配置yarn-env.sh：vim /opt/module/hadoop-2.7.2/etc/hadoop/yarn-env.sh

配置一下JAVA_HOME：export JAVA_HOME=/opt/module/jdk1.8.0_144

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image008-1584383932165.jpg)

（b）配置yarn-site.xml，在<configuration>标签里：

  <!-- Reducer获取数据的方式 -->  <property>      <name>yarn.nodemanager.aux-services</name>      <value>mapreduce_shuffle</value>  </property>     <!-- 指定YARN的ResourceManager的地址 -->  <property>       <name>yarn.resourcemanager.hostname</name>       <value>192.168.247.128</value>  </property>  

（c）配置：mapred-env.sh：vim /opt/module/hadoop-2.7.2/etc/hadoop/mapred-env.sh

配置一下JAVA_HOME：export JAVA_HOME=/opt/module/jdk1.8.0_144

（d）对mapred-site.xml.template重新命名为 mapred-site.xml

命令：cd /opt/module/hadoop-2.7.2/etc/Hadoop

命令：mv mapred-site.xml.template mapred-site.xml

命令：vim mapred-site.xml

在<configuration>标签里配置：

  <!-- 指定MR运行在YARN上 -->  <property>       <name>mapreduce.framework.name</name>       <value>yarn</value>  </property>  

##### 启动集群

（a）启动前必须保证NameNode和DataNode已经启动，jps查看

[root@sire hadoop]# jps

6131 Jps

5478 NameNode

5578 DataNode

（b）启动ResourceManager：[root@sire hadoop]# /opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh start resourcemanager

（c）启动NodeManager：/opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh start nodemanager

查看jps

[root@sire hadoop]# jps

5478 NameNode

6442 NodeManager

6634 Jps

6186 ResourceManager

5578 DataNode

##### 集群操作

（a）浏览器访问http://192.168.247.128:8088/cluster或http://192.168.247.128:8088

 ![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image010-1584383932165.jpg)

（b）删除文件系统上的output文件：/opt/module/hadoop-2.7.2/bin/hdfs dfs -rm -R /user/root/output

（c）执行MapReduce程序：bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /usr/root/input /usr/root/output   

浏览器访问[http://192.168.247.128:8088](http://192.168.247.128:8088/)

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image012-1584383932165.jpg)（d）查看运行结果：bin/hdfs dfs -cat /usr/root/output/*

doop   1

hadoop 1

mapreduce  1

root 2

yarn 1

#### 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

##### 配置mapred-site.xml

命令：vim /opt/module/hadoop-2.7.2/etc/hadoop/mapred-site.xml

在该文件里面<configuration>标签里增加如下配置



  <!-- 历史服务器端地址 -->  <property>    <name>mapreduce.jobhistory.address</name>    <value>192.168.247.128:10020</value>  </property>  <!-- 历史服务器web端地址  -->  <property>      <name>mapreduce.jobhistory.webapp.address</name>      <value>192.168.247.128:19888</value>  </property>  

##### 启动历史服务器

命令：/opt/module/hadoop-2.7.2/sbin/mr-jobhistory-daemon.sh start historyserver

##### 查看历史服务器是否启动

命令：jps

5478 NameNode

7832 Jps

6442 NodeManager

6186 ResourceManager

5578 DataNode

7757 JobHistoryServer

##### 查看JobHistory

浏览器访问：http://192.168.247.128:19888/jobhistory 或 http://192.168.247.128:19888 

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image014-1584383932165.jpg)



#### 配置日志聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

注意：开启日志聚集功能，需要重新启动NodeManager 、ResourceManager和HistoryManager。

开启日志聚集功能具体步骤如下：

##### 配置yarn-site.xml

命令：vim /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml

在该文件里面<configuration>标签里增加如下配置。

  <!-- 日志聚集功能使能 -->  <property>    <name>yarn.log-aggregation-enable</name>    <value>true</value>  </property>     <!-- 日志保留时间设置7天  -->  <property>    <name>yarn.log-aggregation.retain-seconds</name>    <value>604800</value>  </property>  

 

##### 关闭NodeManager 、ResourceManager和HistoryManager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop resourcemanager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh stop historyserver

命令：/opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh stop resourcemanager

stopping resourcemanager

命令：/opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh stop nodemanager

命令：/opt/module/hadoop-2.7.2/sbin/mr-jobhistory-daemon.sh stop historyserver

##### 启动NodeManager 、ResourceManager和HistoryManager

命令：/opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh startresourcemanager

stopping resourcemanager

命令：/opt/module/hadoop-2.7.2/sbin/yarn-daemon.sh start nodemanager

命令：/opt/module/hadoop-2.7.2/sbin/mr-jobhistory-daemon.sh start historyserver

##### 删除HDFS上已经存在的输出文件

命令：/opt/module/hadoop-2.7.2/bin/hdfs dfs -rm -R /usr/root/output

##### 执行WordCount程序

命令：bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /usr/root/input /usr/root/output

##### 查看日志

浏览器访问http://192.168.247.128:19888/jobhistory 

Job History

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image016-1584383932165.jpg)

job运行情况

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image018-1584383932166.jpg)

查看日志

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image020-1584383932166.jpg)

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image022-1584383932166.jpg)

#### 配置文件说明

Hadoop配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值。

##### 默认配置文件

1.在线 http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/SingleCluster.html

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image024-1584383932166.jpg)

2.离线 自己下载相关jar包进行查看

| 要获取的默认文件     | 文件存放在Hadoop的jar包中的位置                             |
| -------------------- | ----------------------------------------------------------- |
| [core-default.xml]   | hadoop-common-2.7.2.jar/  core-default.xml                  |
| [hdfs-default.xml]   | hadoop-hdfs-2.7.2.jar/  hdfs-default.xml                    |
| [yarn-default.xml]   | hadoop-yarn-common-2.7.2.jar/  yarn-default.xml             |
| [mapred-default.xml] | hadoop-mapreduce-client-core-2.7.2.jar/  mapred-default.xml |

##### 自定义配置文件

**core-site.xml****、hdfs-site.xml、yarn-site.xml、mapred-site.xml**四个配置文件存放在$HADOOP_HOME/etc/hadoop这个路径上，用户可以根据项目需求重新进行修改配置。

## 完全分布式运行模式（重点）

分析：

  1）准备3台客户机（关闭防火墙、静态ip、主机名称）

  2）安装JDK

  3）配置环境变量

  4）安装Hadoop

  5）配置环境变量

6）配置集群

7）单点启动

  8）配置ssh

  9）群起并测试集群

#### 虚拟机准备

准备3台客户机（关闭防火墙、静态ip、主机名称）,ip地址分别是

192.168.247.130/131/132

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image026-1584383932166.jpg)

1.关闭防火墙

命令：systemctl stop firewalld.service 

命令：systemctl disable firewalld.service

2.静态ip固定：vim /etc/sysconfig/network-scripts/ifcfg-ens33 

修改：

TYPE=Ethernet

BOOTPROTO=static

NAME=ens33

DEVICE=ens33

ONBOOT=yes

IPADDR=192.168.247.130 #三台机器的ip地址不要设置一致

NETMASK=255.255.255.0

GATEWAY=192.168.247.2

DNS1=114.114.114.114

DNS2=8.8.8.8

重启网络服务器：service network restart

#### 编写集群分发脚本xsync

##### scp（secure copy）安全拷贝

scp定义：scp可以实现服务器与服务器之间的数据拷贝。（from server1 to server2）

基本语法：scp(命令) -r(递归) $pdir/$fname(要拷贝的文件路径/名称) $user@hadoop$host:$pdir/$fname(目的用户@主机:目的路径/名称) 

案例实操

（a）在192.168.247.128上，将192.168.247.128中/opt/module目录下的软件拷贝到192.168.247.130上。

命令：scp -r /opt/module/ root@192.168.247.130:/opt/module/

（b）在192.168.247.132上，将192.168.247.128服务器上的/opt/module目录下的软件拷贝到192.168.247.132上。

命令：scp -r root@192.168.247.128:/opt/module root@192.168.247.131:/opt/module

（c）在192.168.247.132上操作将192.168.247.128中/opt/module目录下的软件拷贝到192.168.247.132上。

命令：scp -r root@192.168.247.128:/opt/module root@192.168.247.132:/opt/module

（d）在192.168.247.128上，将192.168.247.128中/etc/profile文件拷贝到192.168.247.130的/etc/profile上。

命令：scp /etc/profile root@192.168.247.130:/etc/profile

（e）在192.168.247.128上，将192.168.247.128中/etc/profile文件拷贝到192.168.247.131的/etc/profile上。

命令：scp /etc/profile root@192.168.247.131:/etc/profile

（f）在192.168.247.128上，将192.168.247.128中/etc/profile文件拷贝到192.168.247.130的/etc/profile上。

命令：scp /etc/profile root@192.168.247.132:/etc/profile

注意：拷贝过来的配置文件别忘都source一下/etc/profile。

##### rsync远程同步工具

rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

基本语法：

rsync  -rvl    $pdir/$fname       $user@hadoop$host:$pdir/$fname

命令   选项参数  要拷贝的文件路径/名称  目的用户@主机:目的路径/名称

选项参数说明：

| 选项 | 功能         |
| ---- | ------------ |
| -r   | 递归         |
| -v   | 显示复制过程 |
| -l   | 拷贝符号连接 |

案例实操

\1. 在192.168.247.128上，将192.168.247.128上的/opt/software目录同步到192.168.247.130服务器的root用户下的/opt/目录。

命令：rsync -rvl /opt/software/ root@192.168.247.130:/opt/software/

##### xsync集群分发脚本

需求：循环复制文件到所有节点的相同目录下

需求分析：

rsync命令原始拷贝：rsync -rvl /opt/software/ root@192.168.247.130:/opt/software/

期望脚本：

xsync要同步的文件名称，说明：在/root/bin这个目录下存放的脚本，root用户可以在系统任何地方直接执行。

脚本实现

（a）在/root目录下创建bin目录，并在bin目录下xsync创建文件，文件内容如下：

命令：mkdir bin

命令：vim /root/xsync

在该文件中编写如下代码

  #!/bin/bash  #1 获取输入参数个数，如果没有参数，直接退出  pcount=$#  if((pcount==0)); then  echo no args;  exit;  fi     #2 获取文件名称  p1=$1  fname=`basename $p1`  echo fname=$fname     #3 获取上级目录到绝对路径  pdir=`cd -P $(dirname $p1);  pwd`  echo pdir=$pdir     #4 获取当前用户名称  user=`whoami`     #5 循环  for((host=103; host<105;  host++)); do      echo ------------------- hadoop$host  --------------      rsync -rvl $pdir/$fname  $user@hadoop$host:$pdir  done  

（b）修改脚本 xsync 具有执行权限：chmod 777 /root/xsync

（c）调用脚本形式：xsync 文件名称

[atguigu@hadoop102 bin]$ xsync /home/atguigu/bin

注意：如果将xsync放到/home/atguigu/bin目录下仍然不能实现全局使用，可以将xsync移动到/usr/local/bin目录下。

#### 4.3.3 集群配置

\1. 集群部署规划

表2-3

|      | hadoop102          | hadoop103                    | hadoop104                   |
| ---- | ------------------ | ---------------------------- | --------------------------- |
| HDFS | NameNode  DataNode | DataNode                     | SecondaryNameNode  DataNode |
| YARN | NodeManager        | ResourceManager  NodeManager | NodeManager                 |

\2. 配置集群

  （1）核心配置文件

配置core-site.xml

[atguigu@hadoop102 hadoop]$ vi core-site.xml

在该文件中编写如下配置

<!-- 指定HDFS中NameNode的地址 -->

<property>

   <name>fs.defaultFS</name>

   <value>hdfs://hadoop102:9000</value>

</property>

 

<!-- 指定Hadoop运行时产生文件的存储目录 -->

<property>

   <name>hadoop.tmp.dir</name>

   <value>/opt/module/hadoop-2.7.2/data/tmp</value>

</property>

  （2）HDFS配置文件

配置hadoop-env.sh

[atguigu@hadoop102 hadoop]$ vi hadoop-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144

配置hdfs-site.xml

[atguigu@hadoop102 hadoop]$ vi hdfs-site.xml

在该文件中编写如下配置

<property>

   <name>dfs.replication</name>

   <value>3</value>

</property>

 

<!-- 指定Hadoop辅助名称节点主机配置 -->

<property>

   <name>dfs.namenode.secondary.http-address</name>

   <value>hadoop104:50090</value>

</property>

（3）YARN配置文件

配置yarn-env.sh

[atguigu@hadoop102 hadoop]$ vi yarn-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144

配置yarn-site.xml

[atguigu@hadoop102 hadoop]$ vi yarn-site.xml

在该文件中增加如下配置

<!-- Reducer获取数据的方式 -->

<property>

   <name>yarn.nodemanager.aux-services</name>

   <value>mapreduce_shuffle</value>

</property>

 

<!-- 指定YARN的ResourceManager的地址 -->

<property>

   <name>yarn.resourcemanager.hostname</name>

   <value>hadoop103</value>

</property>

（4）MapReduce配置文件

配置mapred-env.sh

[atguigu@hadoop102 hadoop]$ vi mapred-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_144

配置mapred-site.xml

[atguigu@hadoop102 hadoop]$ cp mapred-site.xml.template mapred-site.xml

 

[atguigu@hadoop102 hadoop]$ vi mapred-site.xml

在该文件中增加如下配置

<!-- 指定MR运行在Yarn上 -->

<property>

   <name>mapreduce.framework.name</name>

   <value>yarn</value>

</property>

3．在集群上分发配置好的Hadoop配置文件

[atguigu@hadoop102 hadoop]$ xsync /opt/module/hadoop-2.7.2/

4．查看文件分发情况

[atguigu@hadoop103 hadoop]$ cat /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml

#### 4.3.4 集群单点启动

（1）如果集群是第一次启动，需要**格式化NameNode**

[atguigu@hadoop102 hadoop-2.7.2]$ hadoop namenode -format

（2）在hadoop102上启动NameNode

[atguigu@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start namenode

[atguigu@hadoop102 hadoop-2.7.2]$ jps

3461 NameNode

（3）在hadoop102、hadoop103以及hadoop104上分别启动DataNode

[atguigu@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start datanode

[atguigu@hadoop102 hadoop-2.7.2]$ jps

3461 NameNode

3608 Jps

3561 DataNode

[atguigu@hadoop103 hadoop-2.7.2]$ hadoop-daemon.sh start datanode

[atguigu@hadoop103 hadoop-2.7.2]$ jps

3190 DataNode

3279 Jps

[atguigu@hadoop104 hadoop-2.7.2]$ hadoop-daemon.sh start datanode

[atguigu@hadoop104 hadoop-2.7.2]$ jps

3237 Jps

3163 DataNode

（4）思考：每次都一个一个节点启动，如果节点数增加到1000个怎么办？

  早上来了开始一个一个节点启动，到晚上下班刚好完成，下班？![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image028-1584383932166.jpg)

#### 4.3.5 SSH无密登录配置

\1. 配置ssh

（1）基本语法

ssh另一台电脑的ip地址

（2）ssh连接时出现Host key verification failed的解决方法

[atguigu@hadoop102 opt] $ ssh 192.168.1.103



The authenticity of host '192.168.1.103 (192.168.1.103)' can't be established.

RSA key fingerprint is cf:1e:de:d7:d0:4c:2d:98:60:b4:fd:ae:b1:2d:ad:06.

Are you sure you want to continue connecting (yes/no)? 

Host key verification failed.

（3）解决方案如下：直接输入yes

\2. 无密钥配置

（1）免密登录原理，如图2-40所示

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image030.png)图2-40 免密登陆原理

（2）生成公钥和私钥：

[atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa

然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

（3）将公钥拷贝到要免密登录的目标机器上

[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102

[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103

[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104

注意：

还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104；

还需要在hadoop103上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

\3. .ssh文件夹下（~/.ssh）的文件功能解释

表2-4

| known_hosts     | 记录ssh访问过计算机的公钥(public key) |
| --------------- | ------------------------------------- |
| id_rsa          | 生成的私钥                            |
| id_rsa.pub      | 生成的公钥                            |
| authorized_keys | 存放授权过得无密登录服务器公钥        |

#### 4.3.6 群起集群

\1. 配置slaves

/opt/module/hadoop-2.7.2/etc/hadoop/slaves

[atguigu@hadoop102 hadoop]$ vi slaves

在该文件中增加如下内容：

hadoop102

hadoop103

hadoop104

注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。

同步所有节点配置文件

[atguigu@hadoop102 hadoop]$ xsync slaves

\2. 启动集群

  （1）如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除data和log数据）

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hdfs namenode -format

（2）启动HDFS

[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh

[atguigu@hadoop102 hadoop-2.7.2]$ jps

4166 NameNode

4482 Jps

4263 DataNode

[atguigu@hadoop103 hadoop-2.7.2]$ jps

3218 DataNode

3288 Jps

 

[atguigu@hadoop104 hadoop-2.7.2]$ jps

3221 DataNode

3283 SecondaryNameNode

3364 Jps

（3）启动YARN

[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh

注意：NameNode和ResourceManger如果不是同一台机器，不能在NameNode上启动 YARN，应该在ResouceManager所在的机器上启动YARN。

（4）Web端查看SecondaryNameNode

（a）浏览器中输入：http://hadoop104:50090/status.html

​    （b）查看SecondaryNameNode信息，如图2-41所示。

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image032-1584383932166.jpg)

图2-41 SecondaryNameNode的Web端

\3. 集群基本测试

（1）上传文件到集群

   上传小文件

[atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfs -mkdir -p /user/atguigu/input

[atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfs -put wcinput/wc.input /user/atguigu/input

   上传大文件

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -put

 /opt/software/hadoop-2.7.2.tar.gz  /user/atguigu/input

（2）上传文件后查看文件存放在什么位置

（a）查看HDFS文件存储路径

[atguigu@hadoop102 subdir0]$ pwd

/opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/BP-938951106-192.168.10.107-1495462844069/current/finalized/subdir0/subdir0

（b）查看HDFS在磁盘存储文件内容

[atguigu@hadoop102 subdir0]$ cat blk_1073741825

hadoop yarn

hadoop mapreduce 

atguigu

atguigu

（3）拼接

-rw-rw-r--. 1 atguigu atguigu 134217728 5月 23 16:01 **blk_1073741836**

-rw-rw-r--. 1 atguigu atguigu  1048583 5月 23 16:01 blk_1073741836_1012.meta

-rw-rw-r--. 1 atguigu atguigu 63439959 5月 23 16:01 **blk_1073741837**

-rw-rw-r--. 1 atguigu atguigu  495635 5月 23 16:01 blk_1073741837_1013.meta

[atguigu@hadoop102 subdir0]$ cat blk_1073741836>>tmp.file

[atguigu@hadoop102 subdir0]$ cat blk_1073741837>>tmp.file

[atguigu@hadoop102 subdir0]$ tar -zxvf tmp.file

（4）下载

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -get

 /user/atguigu/input/hadoop-2.7.2.tar.gz ./

#### 4.3.7 集群启动/停止方式总结

\1. 各个服务组件逐一启动/停止

  （1）分别启动/停止HDFS组件

​    hadoop-daemon.sh start / stop namenode / datanode / secondarynamenode

  （2）启动/停止YARN

​    yarn-daemon.sh start / stop resourcemanager / nodemanager

\2. 各个模块分开启动/停止（配置ssh是前提）常用

  （1）整体启动/停止HDFS

​    start-dfs.sh  / stop-dfs.sh

  （2）整体启动/停止YARN

​    start-yarn.sh / stop-yarn.sh

#### 4.3.8 集群时间同步

时间同步的方式：找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，比如，每隔十分钟，同步一次时间。

![img](D:\data\work\mdword\juejin\hadoop\upload\clip_image034.png)

**配置时间同步具体实操：**

\1. 时间服务器配置（必须root用户）

（1）检查ntp是否安装

[**root**@hadoop102 桌面]# rpm -qa|grep ntp

ntp-4.2.6p5-10.el6.centos.x86_64

fontpackages-filesystem-1.41-1.1.el6.noarch

ntpdate-4.2.6p5-10.el6.centos.x86_64

（2）修改ntp配置文件

[**root**@hadoop102 桌面]# vi /etc/ntp.conf

修改内容如下

a）修改1（授权192.168.1.0-192.168.1.255网段上的所有机器可以从这台机器上查询和同步时间）

**#**restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap为

restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

​    b）修改2（集群在局域网中，不使用其他互联网上的时间）

server 0.centos.pool.ntp.org iburst

server 1.centos.pool.ntp.org iburst

server 2.centos.pool.ntp.org iburst

server 3.centos.pool.ntp.org iburst为

**#**server 0.centos.pool.ntp.org iburst

**#**server 1.centos.pool.ntp.org iburst

**#**server 2.centos.pool.ntp.org iburst

**#**server 3.centos.pool.ntp.org iburst

c）添加3（当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步）

server 127.127.1.0

fudge 127.127.1.0 stratum 10

（3）修改/etc/sysconfig/ntpd 文件

[**root**@hadoop102 桌面]# vim /etc/sysconfig/ntpd

增加内容如下（让硬件时间与系统时间一起同步）

SYNC_HWCLOCK=yes

（4）重新启动ntpd服务

[**root**@hadoop102 桌面]# service ntpd status

ntpd 已停

[**root**@hadoop102 桌面]# service ntpd start

正在启动 ntpd：                       [确定]

（5）设置ntpd服务开机启动

[**root**@hadoop102 桌面]# chkconfig ntpd on

\2. 其他机器配置（必须root用户）

（1）在其他机器配置10分钟与时间服务器同步一次

[**root**@hadoop103桌面]# crontab -e

编写定时任务如下：

*/10 * * * * /usr/sbin/ntpdate hadoop102

（2）修改任意机器时间

[**root**@hadoop103桌面]# date -s "2017-9-11 11:11:11"

（3）十分钟后查看机器是否与时间服务器同步

[**root**@hadoop103桌面]# date

说明：测试的时候可以将10分钟调整为1分钟，节省时间。