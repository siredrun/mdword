# hadoop安装

集群规划

| 服务名称 | 子服务            | 服务器  hadoop102 | 服务器  hadoop103 | 服务器  hadoop104 |
| -------- | ----------------- | ----------------- | ----------------- | ----------------- |
| HDFS     | NameNode          | √                 |                   |                   |
|          | DataNode          | √                 | √                 | √                 |
|          | SecondaryNameNode |                   |                   | √                 |
| Yarn     | NodeManager       | √                 | √                 | √                 |
|          | Resourcemanager   |                   | √                 |                   |

## 基本安装

```
sudo yum install -y epel-release psmisc nc net-tools rsync vim lrzsz ntp libzstd openssl-static tree iotop git

cd /opt/software
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
mv /opt/module/hadoop-3.1.3/ /opt/module/hd
sudo vim /etc/profile # 环境变量，在profile文件末尾添加JDK路径：（shitf+g）
#HD_HOME
export HD_HOME=/opt/module/hd
export PATH=$PATH:$HD_HOME/bin
export PATH=$PATH:$HD_HOME/sbin
xsync /opt/module/hd
xsync /etc/profile
source /etc/profile #每个节点都来一下
xcall $HD_HOME/bin/hadoop version # 如显示ERROR: JAVA_HOME is not set and could not be found.暂时不用管它。
```

## hadoop目录

```
bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本
etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
sbin目录：存放启动或停止Hadoop相关服务的脚本
share目录：存放Hadoop的依赖jar包、文档、和官方案例
```

## 配置文件说明

所有的配置文件都在/opt/module/hd/etc/hadoop下。

必须配置

```
core-site.xml
hdfs-site.xml
yarn-site.xml
mapred-site.xml
workers
```

配置Java_home的页面

```
hadoop-env.sh
```

上面三个配置文件直接加入JAVA安装路径即可

```
export JAVA_HOME=/opt/module/java
```

## core-site.xml

```
vim core-site.xml #<configuration>内
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hd/data</value>
    </property>
    <property>
        <name>hadoop.proxyuser.admin.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.admin.groups</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>admin</value>
    </property>
```

## hdfs-site.xml

```
vim hdfs-site.xml #<configuration>内

<!-- SecondaryNameNode：指定Hadoop辅助名称节点主机配置 -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
    <!-- 指定HDFS副本的数量，生产环境下肯定删，测试环境为了提高磁盘工具 -->
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
```

## yarn-site.xml

```
vim yarn-site.xml #<configuration>内
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
```

## mapred-site.xml

```
vim mapred-site.xml #<configuration>内
	<!-- 指定MR运行在Yarn上 -->
	<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
```

## workers

```
vim workers 
hadoop102
hadoop103
hadoop104
```

依次配置完后分发

```
xsync $HD_HOME
```

## 集群启动

如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除所有节点$HD_HOME下的data和log数据）。

```
ssh hadoop102 $HD_HOME/bin/hdfs namenode -format
ssh hadoop102 ls $HD_HOME # 在hadoop102下生成有log和data目录
ssh hadoop102 /opt/module/hd/sbin/start-dfs.sh
Starting namenodes on [hadoop102]
Starting datanodes
hadoop104: WARNING: /opt/module/hd/logs does not exist. Creating.
hadoop103: WARNING: /opt/module/hd/logs does not exist. Creating.
Starting secondary namenodes [hadoop104]
ssh hadoop103 /opt/module/hd/sbin/start-yarn.sh
Starting resourcemanager
Starting nodemanagers
[admin@hadoop104 ~]$ jpsall
========== hadoop102 ==========
4645 NodeManager
4103 DataNode
4776 Jps
3934 NameNode
========== hadoop103 ==========
3569 NodeManager
3058 DataNode
3419 ResourceManager
3934 Jps
========== hadoop104 ==========
3088 DataNode
3218 SecondaryNameNode
3475 NodeManager
3613 Jps
```

访问http://hadoop102:9870/ 

![](https://user-gold-cdn.xitu.io/2020/7/12/17342df14aceaff1?w=1188&h=547&f=png&s=44237)

## lzo编译

基本依赖

```
sudo yum -y install gcc gcc-c++ lzo-devel zlib-devel autoconf automake libtool git maven
vim /etc/maven/settings.xml # 修改sitting.xml加阿里云镜像
	<mirror>  
		<id>alimaven</id>  
		<name>aliyun maven</name>  
		<url>https://maven.aliyun.com/nexus/content/groups/public/</url>  
		<mirrorOf>central</mirrorOf>          
    </mirror>
    
xsync /etc/maven/settings.xml
mvn -v #版本验证
wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
tar -zxvf lzo-2.10.tar.gz
cd lzo-2.10
./configure -prefix=/usr/local/hadoop/lzo/
sudo make
sudo make install
git clone https://github.com/twitter/hadoop-lzo
cd hadoop-lzo
vim pom.xml # 修改Hadoop版本
<hadoop.current.version>3.1.3</hadoop.current.version>

export C_INCLUDE_PATH=/usr/local/hadoop/lzo/include # 声明两个临时环境变量
export LIBRARY_PATH=/usr/local/hadoop/lzo/lib 
mvn package -Dmaven.test.skip=true
ll ../hadoop-lzo/target/ | grep hadoop-lzo-0.4.21-SNAPSHOT.jar # target下即编译成功的hadoop-lzo组件
-rw-rw-r-- 1 admin admin 189001 Jul 12 21:03 hadoop-lzo-0.4.21-SNAPSHOT.jar
```

## hadoop支持lzo压缩

LzopCodec是LzoCodec的升级版（Plus），LzopCodec会压缩的文件中添加一些header信息，保存压缩的元数据。LzoCodec后缀 .lzo_deflate，LzopCodec后缀  .lzo。.lzo_deflate格式的压缩文件，作为MR的输入时，无法使用程序对此文件进行切片；.lzo_deflate格式被Mapper读取时，会乱码。一般都使用且一定使用LzopCodec格式，尤其作为reduce的输出时。在shuffle阶段的话，用哪个都行。

将编译好后的hadoop-lzo-0.4.21-SNAPSHOT.jar 放入/opt/module/hd/share/hadoop/common/

```
cp hadoop-lzo-0.4.21-SNAPSHOT.jar /opt/module/hd/share/hadoop/common/
xsync /opt/module/hd/share/hadoop/common/
vim /opt/module/hd/etc/hadoop/core-site.xml #加入配置
<property>
<name>io.compression.codecs</name>
<value>
org.apache.hadoop.io.compress.GzipCodec,
org.apache.hadoop.io.compress.DefaultCodec,
org.apache.hadoop.io.compress.BZip2Codec,
org.apache.hadoop.io.compress.SnappyCodec,
com.hadoop.compression.lzo.LzoCodec,
com.hadoop.compression.lzo.LzopCodec
</value>
</property>

<property>
    <name>io.compression.codec.lzo.class</name>
    <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>

xsync /opt/module/hd/etc/hadoop/core-site.xml
hd start
jpsall
========== hadoop102 ==========
14983 NodeManager
15256 Jps
14671 DataNode
========== hadoop103 ==========
3584 ResourceManager
3394 DataNode
3726 NodeManager
4175 Jps
========== hadoop104 ==========
2978 DataNode
3093 SecondaryNameNode
3195 NodeManager
3404 Jps

echo hello > hello
hadoop fs -mkdir /wc
hadoop fs -put hello /wc
hadoop fs -ls /wc
Found 1 items
-rw-r--r--   1 admin supergroup          6 2020-07-13 00:34 /wc/hello

# 执行（就是这么长，没有报错表示成功）
hadoop jar /opt/module/hd/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec /wc /wcoutput

hadoop fs -ls /wcoutput #会生成一个/wcoutput/part-r-00000.lzo 后缀lzo的文件
Found 2 items
-rw-r--r--   1 admin supergroup          0 2020-07-13 00:54 /wcoutput/_SUCCESS
-rw-r--r--   1 admin supergroup         58 2020-07-13 00:54 /wcoutput/part-r-00000.lzo

# 为lzo文件创建索引
hadoop jar /opt/module/hd/share/hadoop/common/hadoop-lzo-0.4.21-SNAPSHOT.jar com.hadoop.compression.lzo.DistributedLzoIndexer /wcoutput

hadoop fs -ls /wcoutput # 会增一个*.lzo.index文件。
hadoop fs -ls /wcoutput
Found 3 items
-rw-r--r--   1 admin supergroup          0 2020-07-13 00:54 /wcoutput/_SUCCESS
-rw-r--r--   1 admin supergroup         58 2020-07-13 00:54 /wcoutput/part-r-00000.lzo
-rw-r--r--   1 admin supergroup          8 2020-07-13 00:56 /wcoutput/part-r-00000.lzo.index
```

## 基准测试（重要）

测试HDFS写性能。项目经理问：输入端有1T的数据，问多长时间能把数据上传到集群，假如说1个小时。100T？

```
# 向HDFS集群写10个128M的文件
hadoop jar /opt/module/hd/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

# 下面是测试展示的部分结果
20/06/29 09:57:41 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
20/06/29 09:57:41 INFO fs.TestDFSIO:            Date & time: Mon Jun 29 09:57:41 CST 2020
20/06/29 09:57:41 INFO fs.TestDFSIO:        Number of files: 10 # 文件数量
20/06/29 09:57:41 INFO fs.TestDFSIO: Total MBytes processed: 1280.0 # 文件总大小
20/06/29 09:57:41 INFO fs.TestDFSIO:      Throughput mb/sec: 28.051720359412666 # 吞吐量/每秒 单位：mb
20/06/29 09:57:41 INFO fs.TestDFSIO: Average IO rate mb/sec: 28.580001831054688 # 平均值/每秒 单位：mb
20/06/29 09:57:41 INFO fs.TestDFSIO:  IO rate std deviation: 4.190169301401879 # 标准差 std standard
20/06/29 09:57:41 INFO fs.TestDFSIO:     Test exec time sec: 31.763
```

总结：一秒28mb（大概） 1h大概93600mb 93G；93600*24=2246400 /1024=2193.75 一天能上传2T

测试HDFS读性能

```
# 读取HDFS集群10个128M的文件
hadoop jar /opt/module/hd/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128MB

...
20/06/29 10:08:05 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
20/06/29 10:08:05 INFO fs.TestDFSIO:            Date & time: Mon Jun 29 10:08:05 CST 2020
20/06/29 10:08:05 INFO fs.TestDFSIO:        Number of files: 10
20/06/29 10:08:05 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
20/06/29 10:08:05 INFO fs.TestDFSIO:      Throughput mb/sec: 254.6250248657251
20/06/29 10:08:05 INFO fs.TestDFSIO: Average IO rate mb/sec: 276.227294921875
20/06/29 10:08:05 INFO fs.TestDFSIO:  IO rate std deviation: 83.65365900630701
20/06/29 10:08:05 INFO fs.TestDFSIO:     Test exec time sec: 23.601
20/06/29 10:08:05 INFO fs.TestDFSIO: 
```

总结：一秒260mb，一小时936000mb 936G。

生成文件

```
hadoop fs -ls /benchmarks/TestDFSIO
Found 4 items
drwxr-xr-x   - root supergroup          0 2020-06-29 10:07 /benchmarks/TestDFSIO/io_control
drwxr-xr-x   - root supergroup          0 2020-06-29 09:57 /benchmarks/TestDFSIO/io_data
drwxr-xr-x   - root supergroup          0 2020-06-29 10:08 /benchmarks/TestDFSIO/io_read
drwxr-xr-x   - root supergroup          0 2020-06-29 09:57 /benchmarks/TestDFSIO/io_write
```

删除测试生成数据，hdfs路径/benchmarks文件夹里的内容将被清空

```
hadoop jar  /opt/module/hd/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -clean
```

## 参数调优

1）HDFS参数调优hdfs-site.xml
dfs.namenode.handler.count=20 * log2(Cluster Size)，比如集群规模为8台时，此参数设置为60。8的对数既2的3次方等于8。3*20

```
The number of Namenode RPC server threads that listen to requests from clients. If dfs.namenode.servicerpc-address is not configured then Namenode RPC server threads listen to requests from all nodes.
```

NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作。对于大集群或者有大量客户端的集群来说，通常需要增大参数dfs.namenode.handler.count的默认值10。设置该值的一般原则是将其设置为集群大小的自然对数乘以20，即20logN，N为集群大小。
2）YARN参数调优yarn-site.xml
（1）情景描述：总共7台机器，每天几亿条数据，数据源->Flume->Kafka->HDFS->Hive
面临问题：数据统计主要用HiveSQL，没有数据倾斜，小文件已经做了合并处理，开启的JVM重用，而且IO没有阻塞，内存用了不到50%。但是还是跑的非常慢，而且数据量洪峰过来时，整个集群都会宕掉。基于这种情况有没有优化方案。
（2）解决办法：
内存利用率不够。这个一般是Yarn的2个配置造成的，单个任务可以申请的最大内存大小，和Hadoop单个节点可用内存大小。调节这两个参数能提高系统内存的利用率。
（a）yarn.nodemanager.resource.memory-mb
表示该节点上YARN可使用的物理内存总量，默认是8192（MB），注意，如果你的节点内存资源不够8GB，则需要调减小这个值，而YARN不会智能的探测节点的物理内存总量。
（b）yarn.scheduler.maximum-allocation-mb
单个任务可申请的最多物理内存量，默认是8192（MB）。
3）Hadoop宕机
（1）如果MR造成系统宕机。此时要控制Yarn同时运行的任务数，和每个任务申请的最大内存。调整参数：yarn.scheduler.maximum-allocation-mb（单个任务可申请的最多物理内存量，默认是8192MB）
（2）如果写入文件过量造成NameNode宕机。那么调高Kafka的存储大小，控制从Kafka到HDFS的写入速度。高峰期的时候用Kafka进行缓存，高峰期过去数据同步会自动跟上。

## 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：
1）配置mapred-site.xml

```
vim $HD_HOME/etc/hadoop/mapred-site.xml
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>

xsync $HD_HOME/etc/hadoop/mapred-site.xml
mapred --daemon start historyserver # 访问http://hadoop102:19888/jobhistory jps有jobhistory
```